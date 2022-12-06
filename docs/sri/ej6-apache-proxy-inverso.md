# Ejercicio 6: Apache2 como proxy inverso

## Entrega

### Parte 1
> Entregar configuración de la manera 1

Activo el módulo:
```
sudo a2enmod proxy_http
```

Creo 2 VirtualHosts con las siguientes configuraciones:
```
<VirtualHost *:80>

    ServerName www.app1.org

    ProxyPass "/"  "http://interno.example1.org/"

</VirtualHost>

<VirtualHost *:80>

    ServerName www.app2.org

    ProxyPass "/"  "http://interno.example2.org/"

</VirtualHost>
```

Los habilito:
```
sudo a2ensite app1.conf app2.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

### Parte 2
> Entregar resolución estática para acceder a las páginas de la manera 1

Modifico `/etc/hosts`:
```
# ej6-apache-proxy-inverso
192.168.121.65 www.app1.org www.app2.org
```
Los 2 nombres apuntan a la IP de `proxy`.

### Parte 3
> Capturas del acceso a las páginas como se indica en la manera 1

Muestro que `www.app1.org` funciona:

![](https://i.imgur.com/FdN6R0J.png)

Muestro que `www.app2.org` funciona:

![](https://i.imgur.com/LESHJEN.png)

### Parte 4
> Entregar configuración de la manera 2

Creo 1 VirtualHost con la siguiente configuración:
```
<VirtualHost *:80>

    ServerName www.servidor.org

    ProxyPass "/app1"  "http://interno.example1.org/"
    ProxyPass "/app2"  "http://interno.example2.org/"

</VirtualHost>
```

Lo habilito:
```
sudo a2ensite servidor.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

### Parte 5
> Entregar resolución estática para acceder a las páginas de la manera 2

Modifico `/etc/hosts`:
```
# ej6-apache-proxy-inverso
192.168.121.65 www.servidor.org
```
Ese nombre apunta a la IP de `proxy`.

### Parte 6
> Capturas del acceso a las páginas como se indica en la manera 2

Muestro que `www.servidor.org/app1` funciona:

![](https://i.imgur.com/9gaXPGO.png)

Muestro que `www.servidor.org/app2` funciona:

![](https://i.imgur.com/YZG4kyA.png)








## Ejercicios

### Ejercicio 1
> Descargar `ejercicio_proxy.zip`. Contiene el `Vagrantfile` y el ansible para configurar el escenario.

```
wget https://fp.josedomingo.org/sri2122/u03/doc/ejercicio_proxy/ejercicio_proxy.zip
unzip ejercicio_proxy.zip
```

Muestro la estructura que ha dejado ese zip:
```
atlas@olympus:~/vagrant/ej6-apache-proxy-inverso$ tree
.
├── ansible
│   ├── ansible.cfg
│   ├── group_vars
│   │   └── all
│   ├── hosts
│   ├── roles
│   │   ├── apache2
│   │   │   ├── files
│   │   │   │   ├── index_vhost1.html
│   │   │   │   └── index_vhost2.html
│   │   │   ├── handlers
│   │   │   │   └── main.yaml
│   │   │   ├── tasks
│   │   │   │   └── main.yaml
│   │   │   └── templates
│   │   │       └── etc
│   │   │           └── apache2
│   │   │               └── sites-available
│   │   │                   └── vhost.j2
│   │   └── commons
│   │       └── tasks
│   │           └── main.yaml
│   └── site.yaml
└── Vagrantfile

13 directories, 11 files
```

### Ejercicio 2
> Mostrar el escenario:
>
>- “proxy”:
    - Interfaz eth0 actuando como pública
    - Interfaz en red privada veryisolated
- “servidorweb”:
    - Interfaz conectada a la misma red privada que proxy

Muestro el `Vagrantfile` descargado:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.define :proxy do |proxy|
      proxy.vm.box = "debian/bullseye64"
      proxy.vm.hostname = "proxy"
      proxy.vm.synced_folder ".", "/vagrant", disabled: true
      proxy.vm.network :private_network,
          :libvirt__network_name => "red_privada1",
          :libvirt__dhcp_enabled => false,
          :ip => "10.0.0.10",
          :libvirt__forward_mode => "veryisolated"
      proxy.vm.provision "shell", run: "always", inline: <<-SHELL
        apt-get update && apt upgrade -y
        sysctl -w net.ipv4.ip_forward=1
        iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE
        echo "10.0.0.6 interno.example1.org interno.example2.org" >> /etc/hosts
      SHELL
    end
    config.vm.define :servidorweb do |servidorweb|
      servidorweb.vm.box = "debian/bullseye64"
      servidorweb.vm.hostname = "servidorweb"
      servidorweb.vm.synced_folder ".", "/vagrant", disabled: true
      servidorweb.vm.network :private_network,
          :libvirt__network_name => "red_privada1",
          :libvirt__dhcp_enabled => false,
          :ip => "10.0.0.6",
          :libvirt__forward_mode => "veryisolated"
      servidorweb.vm.provision "shell", run: "always", inline: <<-SHELL
          ip r del default
          ip r add default via 10.0.0.10
      SHELL
    end

  end
```

### Ejercicio 3
> Crear el escenario vagrant

```
vagrant up
```

### Ejercicio 4
> Ejecutar el ansible para configurar “servidorweb”

Modifico el fichero `hosts` con la IP actual de mi “servidorweb”:
```
all:
  children:
    grupo_sw:
      hosts:
        servidor_web:
          ansible_ssh_host: 192.168.121.73
          ansible_ssh_user: vagrant
          ansible_ssh_private_key_file: ../.vagrant/machines/servidorweb/libvirt/private_key
```

Ejecuto el playbook:
```
ansible-playbook site.yaml
```

Este ansible, en la máquina “servidorweb”:

- Instala Apache
- Configura 2 VirtualHosts

### Ejercicio 5
> Instalar Apache en "proxy"

```
sudo apt install apache2
```

### Ejercicio 6
>Configurar "proxy" para acceder a las páginas de “servidorweb”...
>
>    - Manera 1: acceder a la primera en `www.app1.org` y a la segunda en `www.app2.org`

#### En mi máquina

Modifico `/etc/hosts`:
```
# ej6-apache-proxy-inverso
192.168.121.65 www.app1.org www.app2.org
```
Los 2 nombres apuntan a la IP de `proxy`.

#### En proxy

Activo el módulo:
```
sudo a2enmod proxy_http
```

Creo 2 VirtualHosts con las siguientes configuraciones:
```
<VirtualHost *:80>

    ServerName www.app1.org

    ProxyPass "/"  "http://interno.example1.org/"

</VirtualHost>

<VirtualHost *:80>

    ServerName www.app2.org

    ProxyPass "/"  "http://interno.example2.org/"

</VirtualHost>
```

Los habilito:
```
sudo a2ensite app1.conf app2.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Muestro que `www.app1.org` funciona:

![](https://i.imgur.com/FdN6R0J.png)

Muestro que `www.app2.org` funciona:

![](https://i.imgur.com/LESHJEN.png)

### Ejercicio 7
>Configurar "proxy" para acceder a las páginas de “servidorweb”...
>
>    - Manera 2: acceder a la primera en `www.servidor.org/app1` y a la segunda en `www.servidor.org/app2`

#### En mi máquina

Modifico `/etc/hosts`:
```
# ej6-apache-proxy-inverso
192.168.121.65 www.servidor.org
```
Ese nombre apunta a la IP de `proxy`.

#### En proxy

Creo 1 VirtualHost con la siguiente configuración:
```
<VirtualHost *:80>

    ServerName www.servidor.org

    ProxyPass "/app1"  "http://interno.example1.org/"
    ProxyPass "/app2"  "http://interno.example2.org/"

</VirtualHost>
```

Lo habilito:
```
sudo a2ensite servidor.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Muestro que `www.servidor.org/app1` funciona:

![](https://i.imgur.com/9gaXPGO.png)

Muestro que `www.servidor.org/app2` funciona:

![](https://i.imgur.com/YZG4kyA.png)
