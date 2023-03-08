# LDAPs

## 1. Enunciado

Configura el servidor LDAP de alfa para que utilice el protocolo ldaps:// a la vez que el ldap:// utilizando el certificado x509 de la práctica de https o solicitando el correspondiente a través de gestiona. Realiza las modificaciones adecuadas en los clientes ldap de alfa para que todas las consultas se realicen por defecto utilizando ldaps://

## 2. Server

### 2.1 Autoridad certificadora

```shell
sudo apt install easy-rsa
mkdir ~/easy-rsa
ln -s /usr/share/easy-rsa/* ~/easy-rsa/
chmod 700 /home/vagrant/easy-rsa
cd ~/easy-rsa
./easyrsa init-pki
nano vars
```

```shell
set_var EASYRSA_REQ_COUNTRY    "ES"
set_var EASYRSA_REQ_PROVINCE   "Sevilla"
set_var EASYRSA_REQ_CITY       "Dos Hermanas"
set_var EASYRSA_REQ_ORG        "IES Gonzalo Nazareno"
set_var EASYRSA_REQ_EMAIL      "adrjaro@gmail.com"
set_var EASYRSA_REQ_OU         "ASIR"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
```

```shell
./easyrsa build-ca
```

Password: 1234
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:server.adrianj.gonzalonazareno.org

Importo el certificado:

```shell
sudo cp ~/easy-rsa/pki/ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

### 2.2 CSR servidor y firmado

```shell
mkdir ~/server-csr
cd ~/server-csr
openssl genrsa -out server.key
```

```shell
vagrant@server:~/server-csr$ openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Sevilla
Locality Name (eg, city) []:Dos Hermanas
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Gonzalo Nazareno
Organizational Unit Name (eg, section) []:ASIR
Common Name (e.g. server FQDN or YOUR name) []:server.adrianj.gonzalonazareno.org
Email Address []:adrjaro@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

```shell
cd ~/easy-rsa
./easyrsa import-req ~/server-csr/server.csr server
./easyrsa sign-req server server
cp pki/issued/server.crt ~/server-csr/
cd ~/server-csr/
```

Copio los ficheros al directorio necesario:

```shell
sudo cp /home/vagrant/server-csr/server.key /home/vagrant/server-csr/server.crt /etc/ssl/certs/ca-certificates.crt /etc/ldap/sasl2/
```

Modifico propietarios:

```shell
sudo chown openldap:openldap /etc/ldap/sasl2/server.key /etc/ldap/sasl2/server.crt /etc/ldap/sasl2/ca-certificates.crt
```














### 2.3 Configuración certificados

```shell
sudo su -
nano mod_ssl.ldif
```

```shell
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ldap/sasl2/ca-certificates.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/sasl2/server.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/sasl2/server.key
```

La aplico:

```shell
ldapmodify -Y EXTERNAL -H ldapi:/// -f mod_ssl.ldif
```

Compruebo que se ha aplicado:

```shell
slapcat -b cn=config
```

![certificadosaplicados](https://i.imgur.com/I5s3MNw.png)

### 2.3 Configuración ldap

Reemplazo la siguiente línea:

```shell
sudo nano /etc/default/slapd
```

```shell
SLAPD_SERVICES="ldap:/// ldapi:/// ldaps:///"
```

Reinicio:

```shell
sudo systemctl restart slapd
```

### 2.4 Comprobaciones

Puerto de LDAPs abierto:

```shell
sudo ss -tulpn | grep 636
```

![ldapspuerto](https://i.imgur.com/WXkopWr.png)
























## 3. Clientedebian

### 3.1 Configuración

Reemplazo las siguientes líneas:

```shell
sudo nano /etc/nslcd.conf
```

```shell
# SSL options
ssl start_tls
tls_reqcert never
# tls_cacertfile /etc/ssl/certs/ca-certificates.crt
```

Reinicio:

```shell
sudo systemctl restart nslcd
```



























