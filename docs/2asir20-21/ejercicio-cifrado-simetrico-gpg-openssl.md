---
title: "Ejercicio: Cifrado simétrico con gpg+openssl"
---

# 1. Crea un documento de texto con cualquier editor o utiliza uno del que dispongas

```
touch para_miguel.txt
```
Contenido de ese fichero...
```
vagrant@ejercicio-simetrico-gpg:~$ cat para_miguel.txt
si lo ves lo has descrifrado
```


# 2. Cifra este documento con alguna contraseña acordada con el compañero de al lado

```
gpg -c para_miguel.txt
```

> **Contraseña usada al cifrar**: hola

Se genera...
```
para_miguel.txt.gpg
```

# 3. Haz llegar por algún medio al compañero de al lado el documento que acabas de cifrar

Nos pasamos los ficheros cifrados por chat privado en slack

# 4. Descifra el documento que te ha hecho llegar tu compañero de al lado

```
gpg -d fichero1.gpg
```

> **Contraseña usada al descifrar**: hola

Resultado...
```
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
hola mundo
```

# 5. ...

## 5.1 ¿Con qué algoritmo se ha cifrado el fichero?

El siguiente comando muestra toda la información gpg relacionada con el cifrado

```
gpg --list-packets para_miguel.txt.gpg
```
> ¡Se nos **pide la frase de paso** para realizar ésta acción!


La salida completa es...
```
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
# off=0 ctb=8c tag=3 hlen=2 plen=13
:symkey enc packet: version 4, cipher 9, s2k 3, hash 2
	salt A7E12EC5F2161D3A, count 65011712 (255)
# off=15 ctb=d2 tag=18 hlen=2 plen=95 new-ctb
:encrypted data packet:
	length: 95
	mdc_method: 2
# off=36 ctb=a3 tag=8 hlen=1 plen=0 indeterminate
:compressed packet: algo=1
# off=38 ctb=ac tag=11 hlen=2 plen=50
:literal data packet:
	mode b (62), created 1602068487, name="para_miguel.txt",
	raw data: 29 bytes
```
...pero nos interesa que al inicio, nos indica el algoritmo de cifrado usado para generar el fichero con la línea:
```
gpg: AES256 encrypted data
```


## 5.2 Vuelve a cifrar el fichero usando el algoritmo AES256
```
gpg -c --cipher-algo AES256 para_miguel_AES256.txt 
```
> Como dato añadido, actualmente al cifrar simétricamente sin indicar un algoritmo, por defecto lo hace con **AES256**.  
En el primer fichero que cifré en este ejercicio, no marcamos algoritmo, así que supuso AES256.  
Marcar ahora mismo ese algoritmo para cifrar tendría poco sentido, ya que no haría falta...A no ser que nuestro algoritmo por defecto fuese otro.   


Si queremos comprobar que de verdad se aplicó ese cifrado...
```
gpg --list-packets para_miguel_AES256.txt.gpg
```
```
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
# off=0 ctb=8c tag=3 hlen=2 plen=13
:symkey enc packet: version 4, cipher 9, s2k 3, hash 2
	salt C4A88339D9DEC6C4, count 65011712 (255)
# off=15 ctb=d2 tag=18 hlen=2 plen=102 new-ctb
:encrypted data packet:
	length: 102
	mdc_method: 2
# off=36 ctb=a3 tag=8 hlen=1 plen=0 indeterminate
:compressed packet: algo=1
# off=38 ctb=ac tag=11 hlen=2 plen=57
:literal data packet:
	mode b (62), created 1602071080, name="para_miguel_AES256.txt",
	raw data: 29 bytes
```
En efecto, se aplicó.

## 5.3 ¿Puedes hacer permanente ésta configuración?

Toda configuración permanente de gpg se encuentra en la ruta `$HOME/.gnupg/gpg.conf`. Por defecto este fichero no se crea, así que lo creamos
```
touch $HOME/.gnupg/gpg.conf
```

En ese fichero, para establecer un algoritmo de cifrado simétrico por defecto que nosotros queramos, escribimos lo siguiente 
```
personal-cipher-preferences AES256
```
A partir de ahora, cada vez que cifremos simétricamente, si no especificamos explícitamente otro algoritmo, éste será el utilizado



# 6. Instala gpg en windows (Gpg4win), repite el ejercicio en Windows. Puedes encriptar un mensaje en linux y desencriptarlo en windows y viceversa

[Descarga aquí Gpg4win](https://www.gpg4win.org/download.html)

## Encriptación en Windows - Desencriptación en Linux
Encripto en Windows con (desde cmd)...
```
gpg -c para_linux.txt
```
> Usaré la misma frase de paso que antes, **hola**

Se me genera un `para_linux.txt.gpg`, el fichero cifrado.  
Procedo a pasarlo a Linux para su desencriptación.

Una vez lo tengo, desencripto de la misma manera que hicimos anteriormente
```
gpg -d para_linux.txt.gpg
```
```
gpg: AES encrypted data
gpg: encrypted with 1 passphrase
fichero a descifrar en linux
```
Funcionó correctamente


## Encriptación en Linux - Desencriptación en Windows
Encripto en Linux con...
```
gpg -c para_windows.txt
```
> Misma frase de paso, **hola**

Se genera `para_windows.txt.gpg`, el fichero cifrado.  
Procedo a pasarlo a Windows para su desencriptación.

Una vez lo tengo, desencripto...
![](https://i.imgur.com/OPCJS24.png)

Funcionó correctamente


# 7. `openssl` es otra herramienta que nos permite cifrar mensajes de forma simétrica, investiga cómo se realiza este ejercicio utilizando esta herramienta

## Proceso de cifrado
```
openssl enc -v -aes-256-cbc -in con_openssl.txt -out con_openssl.txt.enc
```
> Misma frase de paso, **hola**. Introduzco el parámetro `-v` para que me muestre información detallada del proceso, pero no sería obligatorio usarlo

```
bufsize=8192
enter aes-256-cbc encryption password:
Verifying - enter aes-256-cbc encryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bytes read   :       52
bytes written:       80
```
Después de esto ya tendríamos generado nuestro fichero cifrado `con_openssl.txt.enc`


## Proceso de descifrado
```
openssl enc -v -aes-256-cbc -d -in con_openssl.txt.enc -out con_openssl_descifrado.txt
```
```
bufsize=8192
enter aes-256-cbc decryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bytes read   :       80
bytes written:       52
```
Ésta operación nos ha generado nuestro fichero desencriptado `con_openssl_descifrado.txt`  

Procedemos a ver su contenido
```
vagrant@ejercicio-simetrico-gpg:~$ cat con_openssl_descifrado.txt
Has descifrado correctamente el fichero con openssl
```


