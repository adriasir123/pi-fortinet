# Ejercicio 1: Instalación y configuración del servidor bind9 en nuestra red local

## Entrega

### Parte 1
> Configuración del cliente, para comprobar que consulta al servidor DNS que hemos configurado.

modificar /etc/resolv.conf

### Parte 2
> Entrega las zonas que has definido.


### Parte 3
> Realiza las siguientes consultas dig/nslookup desde el cliente:
>
>- Dirección de tunombre.iesgn.org, www.iesgn.org, ftp.iesgn.org
>- El servidor DNS con autoridad sobre la zona del dominio iesgn.org
>- El servidor de correo configurado para iesgn.org
>- La dirección IP de www.josedomingo.org
>- Una resolución inversa









## Escenario

### Ejercicio 1
> Crear el escenario

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :server do |server|
    server.vm.box = "debian/bullseye64"
    server.vm.hostname = "server"
    server.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__forward_mode => "veryisolated"
  end

end
```

### Ejercicio 2
> Instalar en `server` un Apache y configurar 2 VirtualHosts: `www.iesgn.org` y `departamentos.iesgn.org`

```
sudo apt update
sudo apt install apache2
```

En `www.iesgn.org.conf`:
```
<VirtualHost *:80>

	ServerName www.iesgn.org

	DocumentRoot /var/www/www

</VirtualHost>
```

En `departamentos.iesgn.org.conf`:
```
<VirtualHost *:80>

	ServerName departamentos.iesgn.org

	DocumentRoot /var/www/departamentos

</VirtualHost>
```

Los habilito y reinicio:
```
sudo a2ensite www.iesgn.org.conf departamentos.iesgn.org.conf
sudo systemctl restart apache2
```




2. Vamos a instalar en nuestra red local un servidor DNS (lo puedes instalar en el mismo equipo que tiene el servidor web)

3. El nombre del servidor DNS va a ser adrianj.iesgn.org.

cambiar hostname, cambiar /etc/hosts


















## Servidor bind9

Instala un servidor dns bind9. Las características del servidor DNS que queremos instalar son las siguientes:


- El servidor DNS se llama tunombre.iesgn.org y por supuesto, va a ser el servidor con autoridad para la zona iesgn.org.

- Vamos a suponer que tenemos un servidor para recibir los correos que se llame correo.iesgn.org y que está en la dirección x.x.x.200 (esto es ficticio).

- Vamos a suponer que tenemos un servidor ftp que se llame ftp.iesgn.org y que está en x.x.x.201 (esto es ficticio)

- Además queremos nombrar a los clientes.

nombro al cliente que tengo y 1 o 2 más ficticios

- También hay que nombrar a los virtual hosts de apache: www.iesgn.org y departamentos.iesgn.org

- Se tienen que configurar la zona de resolución inversa.
