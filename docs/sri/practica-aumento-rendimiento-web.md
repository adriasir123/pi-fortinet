# Práctica: Aumento de rendimiento en servidores web

## HAProxy: Balanceador de carga

### Parte 1

> Crear y configurar el escenario usando el repositorio [vagrant_ansible_haproxy](https://github.com/josedom24/vagrant_ansible_haproxy)

Clono el repo:

```console
git clone https://github.com/josedom24/vagrant_ansible_haproxy.git
```

Creo las máquinas:

```console
vagrant up
```

Entro en cada una de ellas y apunto sus IPs:

* frontend: 192.168.121.61
* backend1: 192.168.121.196
* backend2: 192.168.121.20

Modifico `ansible/hosts` con estas IPs:

```console
[servidor_ha]
frontend ansible_ssh_host=192.168.121.61 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/frontend/libvirt/private_key ansible_python_interpreter=/usr/bin/python3

[servidores_web]
backend1 ansible_ssh_host=192.168.121.196 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/backend1/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
backend2 ansible_ssh_host=192.168.121.20 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/backend2/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
```

Ejecuto el playbook para configurar el escenario:

```console
ansible-playbook site.yaml
```

Podemos llegar a encontrarnos con este fallo, y lo documento para que se sepa:

![fallo ansible https](https://i.imgur.com/Lhcq8bQ.png)

Para arreglarlo tendríamos que entrar en cada máquina y cambiar los repositorios en `/etc/apt/sources.list` a https:

![cambio a https](https://i.imgur.com/jPufWsl.png)

Una vez hecho esto, vuelvo a ejecutar el playbook y ya funcionaría:

```console
atlas@olympus:~/vagrant/vagrant_ansible_haproxy/ansible$ ansible-playbook site.yaml

PLAY [all] ************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [frontend]
ok: [backend1]
ok: [backend2]

TASK [commons : Ensure system is updated] *****************************************************************************************************************
ok: [backend2]
ok: [backend1]
ok: [frontend]

PLAY [servidor_ha] ****************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [frontend]

TASK [haproxy : install haproxy] **************************************************************************************************************************
changed: [frontend]

PLAY [servidores_web] *************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [backend2]
ok: [backend1]

TASK [nginx : install nginx, php-fpm] *********************************************************************************************************************
changed: [backend1]
changed: [backend2]

TASK [nginx : Copy info.php] ******************************************************************************************************************************
changed: [backend1]
changed: [backend2]

TASK [nginx : Copy virtualhost default] *******************************************************************************************************************
changed: [backend1]
changed: [backend2]

RUNNING HANDLER [nginx : restart nginx] *******************************************************************************************************************
changed: [backend2]
changed: [backend1]

PLAY [servidores_web] *************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [backend1]
ok: [backend2]

TASK [mariadb : ensure mariadb is installed] **************************************************************************************************************
changed: [backend1]
changed: [backend2]

TASK [mariadb : ensure mariadb binds to internal interface] ***********************************************************************************************
changed: [backend2]
changed: [backend1]

RUNNING HANDLER [mariadb : restart mariadb] ***************************************************************************************************************
changed: [backend2]
changed: [backend1]

PLAY [servidores_web] *************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [backend1]
ok: [backend2]

TASK [wordpress : install unzip] **************************************************************************************************************************
changed: [backend2]
changed: [backend1]

TASK [wordpress : download wordpress] *********************************************************************************************************************
changed: [backend2]
changed: [backend1]

TASK [wordpress : unzip wordpress] ************************************************************************************************************************
changed: [backend2]
changed: [backend1]

TASK [wordpress : Copy wordpress.sql] *********************************************************************************************************************
changed: [backend1]
changed: [backend2]

TASK [wordpress : create database wordpress] **************************************************************************************************************
changed: [backend2]
changed: [backend1]

TASK [wordpress : create user mysql wordpress] ************************************************************************************************************
changed: [backend1] => (item=localhost)
changed: [backend2] => (item=localhost)

TASK [wordpress : copy wp-config.php] *********************************************************************************************************************
changed: [backend1]
changed: [backend2]

RUNNING HANDLER [wordpress : restart nginx] ***************************************************************************************************************
changed: [backend1]
changed: [backend2]

PLAY RECAP ************************************************************************************************************************************************
backend1                   : ok=20   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
backend2                   : ok=20   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
frontend                   : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Parte 2

> Configurar HAProxy en frontend

Al final de `/etc/haproxy/haproxy.cfg` añado:

```console
frontend servidores_web
	bind *:80
	mode http
	stats enable
	stats uri /ha_stats
	stats auth  cda:cda
	default_backend servidores_web_backend

backend servidores_web_backend
	mode http
	balance roundrobin
	server backend1 10.0.0.10:80 check
	server backend2 10.0.0.11:80 check
```

Reinicio HAProxy:

```console
sudo systemctl restart haproxy
```

Compruebo que ha empezado a escuchar en el puerto 80:

```console
vagrant@frontend:/etc/haproxy$ sudo lsof -i -P -n | grep LISTEN
sshd       524    root    3u  IPv4  12470      0t0  TCP *:22 (LISTEN)
sshd       524    root    4u  IPv6  12481      0t0  TCP *:22 (LISTEN)
haproxy  20027 haproxy    7u  IPv4  41481      0t0  TCP *:80 (LISTEN)
```

### Parte 3

> Configurar la resolución estática y acceder a wordpress

`/etc/hosts`:

```console
# haproxy
192.168.121.61 www.example.org
```

![wordpress haproxy](https://i.imgur.com/PHC8eCO.png)

En cada nodo tenemos un `info.php`, así que podemos comprobar si el balanceo funciona mirando los hostnames:

![info.php backend1](https://i.imgur.com/lIZh0TW.png)

![info.php backend2](https://i.imgur.com/SNEh1Qa.png)

### Parte 4

> Calcular el rendimiento haciendo 5 pruebas. Obtener la media de `Requests per second`

Hago una primera prueba con:

```console
ab -t 10 -c 100 http://www.example.org/wordpress/
```

Estos son los resultados:

```console
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.example.org (be patient)
Finished 917 requests


Server Software:        nginx/1.18.0
Server Hostname:        www.example.org
Server Port:            80

Document Path:          /wordpress/
Document Length:        26892 bytes

Concurrency Level:      100
Time taken for tests:   10.009 seconds
Complete requests:      917
Failed requests:        508
   (Connect: 0, Receive: 0, Length: 508, Exceptions: 0)
Total transferred:      24881439 bytes
HTML transferred:       24672135 bytes
Requests per second:    91.62 [#/sec] (mean)
Time per request:       1091.467 [ms] (mean)
Time per request:       10.915 [ms] (mean, across all concurrent requests)
Transfer rate:          2427.70 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.5      0       2
Processing:    14  954 945.3    559    2454
Waiting:       10  933 937.0    539    2427
Total:         14  954 945.2    559    2454

Percentage of the requests served within a certain time (ms)
  50%    556
  66%   1356
  75%   2086
  80%   2279
  90%   2389
  95%   2399
  98%   2418
  99%   2436
 100%   2454 (longest request)
```

La parte que nos importa:

```console
Requests per second:    91.62 [#/sec] (mean)
```

Hago 4 pruebas más y saco una media:

```console
Requests per second:    102.89 [#/sec] (mean)
Requests per second:    101.58 [#/sec] (mean)
Requests per second:    101.68 [#/sec] (mean)
Requests per second:    99.28 [#/sec] (mean)
```

![media rendimiento haproxy](https://i.imgur.com/i9PUvNo.png)

### Parte 5

#### 5.1

> Instalar `hatop`

```console
sudo apt install hatop
```

#### 5.2

> Acceder a las estadísticas/administración

```console
sudo hatop -s /run/haproxy/admin.sock
```

![hatop console](https://i.imgur.com/33AUzSO.png)

#### 5.3

> Deshabilitar el nodo backend1

Pulso F10 encima del nodo para que entre en mantenimiento:

![backend1 maint](https://i.postimg.cc/528W3cw6/backend1-maint.gif)

#### 5.4

> Volver a hacer las pruebas de rendimiento

```console
Requests per second:    43.26 [#/sec] (mean)
Requests per second:    38.94 [#/sec] (mean)
Requests per second:    45.58 [#/sec] (mean)
Requests per second:    45.73 [#/sec] (mean)
Requests per second:    45.78 [#/sec] (mean)
```

![media rendimiento haproxy 1 nodo](https://i.imgur.com/C5Em08S.png)

#### 5.5

> ¿Se nota la diferencia entre balancear y no balancear?

Sí, básicamente porque al tener un solo nodo el rendimiento baja a la mitad *(lógicamente)*.

#### 5.6

> Habilitar de nuevo el nodo backend1

Hago lo mismo que para deshabilitarlo, pero esta vez pulso F9:

![backend1 up](https://i.postimg.cc/0QW93b9V/backend1-up.gif)

### Parte 6

Añadir un nuevo nodo backend3 donde se instale Wordpress.

#### 6.1

> Modificar el Vagrantfile

```console
    config.vm.define :backend3 do |backend3|
      backend3.vm.box = "debian/bullseye64"
      backend3.vm.hostname = "backend3"
      backend3.vm.synced_folder ".", "/vagrant", disabled: true
      backend3.vm.network :private_network,
        :libvirt__network_name => "red1",
        :libvirt__dhcp_enabled => false,
        :ip => "10.0.0.12",
        :libvirt__forward_mode => "veryisolated"
    end
```

Creo la máquina:

```console
vagrant up backend3
```

#### 6.2

> Modificar el Ansible

`ansible/hosts`:

```console
[servidor_ha]
frontend ansible_ssh_host=192.168.121.61 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/frontend/libvirt/private_key ansible_python_interpreter=/usr/bin/python3

[servidores_web]
backend1 ansible_ssh_host=192.168.121.196 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/backend1/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
backend2 ansible_ssh_host=192.168.121.20 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/backend2/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
backend3 ansible_ssh_host=192.168.121.236 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/backend3/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
```

Ejecuto el playbook de nuevo para configurar la nueva máquina como las demás:

```console
atlas@olympus:~/vagrant/vagrant_ansible_haproxy/ansible$ ansible-playbook site.yaml

PLAY [all] *************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [backend2]
ok: [frontend]
ok: [backend3]
ok: [backend1]

TASK [commons : Ensure system is updated] ******************************************************************************************************************************
ok: [backend2]
ok: [backend1]
ok: [frontend]
changed: [backend3]

PLAY [servidor_ha] *****************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [frontend]

TASK [haproxy : install haproxy] ***************************************************************************************************************************************
ok: [frontend]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [backend3]
ok: [backend1]
ok: [backend2]

TASK [nginx : install nginx, php-fpm] **********************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

TASK [nginx : Copy info.php] *******************************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

TASK [nginx : Copy virtualhost default] ********************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

RUNNING HANDLER [nginx : restart nginx] ********************************************************************************************************************************
changed: [backend3]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [backend1]
ok: [backend2]
ok: [backend3]

TASK [mariadb : ensure mariadb is installed] ***************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

TASK [mariadb : ensure mariadb binds to internal interface] ************************************************************************************************************
ok: [backend2]
ok: [backend1]
changed: [backend3]

RUNNING HANDLER [mariadb : restart mariadb] ****************************************************************************************************************************
changed: [backend3]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [backend1]
ok: [backend2]
ok: [backend3]

TASK [wordpress : install unzip] ***************************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

TASK [wordpress : download wordpress] **********************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

TASK [wordpress : unzip wordpress] *************************************************************************************************************************************
changed: [backend3]
ok: [backend1]
ok: [backend2]

TASK [wordpress : Copy wordpress.sql] **********************************************************************************************************************************
changed: [backend1]
changed: [backend2]
changed: [backend3]

TASK [wordpress : create database wordpress] ***************************************************************************************************************************
changed: [backend3]
changed: [backend2]
changed: [backend1]

TASK [wordpress : create user mysql wordpress] *************************************************************************************************************************
changed: [backend3] => (item=localhost)
ok: [backend2] => (item=localhost)
ok: [backend1] => (item=localhost)

TASK [wordpress : copy wp-config.php] **********************************************************************************************************************************
ok: [backend1]
ok: [backend2]
changed: [backend3]

RUNNING HANDLER [wordpress : restart nginx] ****************************************************************************************************************************
changed: [backend3]

PLAY RECAP *************************************************************************************************************************************************************
backend1                   : ok=17   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
backend2                   : ok=17   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
backend3                   : ok=20   changed=16   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
frontend                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### 6.3

> Añadir backend3 a `/etc/haproxy/haproxy.cfg` para que se tome en cuenta para el balanceo

```console
backend servidores_web_backend
	mode http
	balance roundrobin
	server backend1 10.0.0.10:80 check
	server backend2 10.0.0.11:80 check
	server backend3 10.0.0.12:80 check
```

Reinicio HAProxy:

```console
sudo systemctl restart haproxy
```

#### 6.4

> Volver a hacer las pruebas de rendimiento

```console
Requests per second:    130.12 [#/sec] (mean)
Requests per second:    129.43 [#/sec] (mean)
Requests per second:    129.21 [#/sec] (mean)
Requests per second:    129.64 [#/sec] (mean)
Requests per second:    130.01 [#/sec] (mean)
```

![media rendimiento haproxy 3 nodos](https://i.imgur.com/YYRUC3h.png)

#### 6.5

> ¿Se nota la diferencia al balancear añadiendo un nuevo nodo?

Lógicamente sí. El rendimiento aumenta proporcionalmente.

## Memcached

### 2.1

> Crear y configurar el escenario [vagrant_ansible_wordpress](https://github.com/josedom24/vagrant_ansible_wordpress.git)

Clono el repo:

```console
git clone https://github.com/josedom24/vagrant_ansible_wordpress.git
```

Creo la máquina:

```console
vagrant up
```

Entro en ella y apunto su IP: 192.168.121.239

Modifico `ansible/hosts` con esta IP:

```console
[servidores_web]
servidor_web ansible_ssh_host=192.168.121.239 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/default/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
```

Ejecuto el playbook para configurar la máquina:

```console
atlas@olympus:~/vagrant/vagrant_ansible_wordpress/ansible$ ansible-playbook site.yaml
[WARNING]: ansible.utils.display.initialize_locale has not been called, this may result in incorrectly calculated text widths that can cause Display to print incorrect
line lengths

PLAY [all] *************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [servidor_web]

TASK [commons : Ensure system is updated] ******************************************************************************************************************************
changed: [servidor_web]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [servidor_web]

TASK [nginx : install nginx, php-fpm] **********************************************************************************************************************************
changed: [servidor_web]

TASK [nginx : Copy info.php] *******************************************************************************************************************************************
changed: [servidor_web]

TASK [nginx : Copy virtualhost default] ********************************************************************************************************************************
changed: [servidor_web]

RUNNING HANDLER [nginx : restart nginx] ********************************************************************************************************************************
changed: [servidor_web]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [servidor_web]

TASK [mariadb : ensure mariadb is installed] ***************************************************************************************************************************
changed: [servidor_web]

TASK [mariadb : ensure mariadb binds to internal interface] ************************************************************************************************************
changed: [servidor_web]

RUNNING HANDLER [mariadb : restart mariadb] ****************************************************************************************************************************
changed: [servidor_web]

PLAY [servidores_web] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [servidor_web]

TASK [wordpress : install unzip] ***************************************************************************************************************************************
changed: [servidor_web]

TASK [wordpress : download wordpress] **********************************************************************************************************************************
changed: [servidor_web]

TASK [wordpress : unzip wordpress] *************************************************************************************************************************************
changed: [servidor_web]

TASK [wordpress : Copy wordpress.sql] **********************************************************************************************************************************
changed: [servidor_web]

TASK [wordpress : create database wordpress] ***************************************************************************************************************************
changed: [servidor_web]

TASK [wordpress : create user mysql wordpress] *************************************************************************************************************************
changed: [servidor_web] => (item=localhost)

TASK [wordpress : copy wp-config.php] **********************************************************************************************************************************
changed: [servidor_web]

RUNNING HANDLER [wordpress : restart nginx] ****************************************************************************************************************************
changed: [servidor_web]

PLAY RECAP *************************************************************************************************************************************************************
servidor_web               : ok=20   changed=16   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.2

> Configurar la resolución estática

`/etc/hosts`:

```console
# memcached
192.168.121.239 www.example.org
```

### 2.3

> Instalar memcached. Comprobar en `info.php` que funciona.

```console
sudo apt install php-memcached memcached
```

Si se instaló correctamente, nos aparece esta sección:

![memcached funciona](https://i.imgur.com/yTbr56w.png)

### 2.4

> Instalar y configurar W3 Total Cache en Wordpress para que se pueda comunicar con memcached

![W3 Total Cache](https://i.imgur.com/GVh1pjM.png)

Comienzo la configuración de forma manual:

![w3 config 1](https://i.imgur.com/qsRIamj.png)

En `General Settings` hay distintas cachés y opciones que podemos activar y configurar con este plugin. Lo hago:

![page cache](https://i.imgur.com/WjEn4mo.png)

![Minify](https://i.imgur.com/Bhvp13A.png)

![Database Cache](https://i.imgur.com/dOnOOSX.png)

![Object Cache](https://i.imgur.com/SHu0coP.png)

![Fragment Cache Method](https://i.imgur.com/71WO6eu.png)

Guardo las configuraciones:

![save settings](https://i.imgur.com/KD1hGVl.png)

Reinicio Nginx:

```console
sudo systemctl restart nginx
```

### 2.5

> Realizar el cálculo de rendimiento y obtener la media

```console
Requests per second:    1083.49 [#/sec] (mean)
Requests per second:    991.49 [#/sec] (mean)
Requests per second:    1075.94 [#/sec] (mean)
Requests per second:    1089.48 [#/sec] (mean)
Requests per second:    1088.40 [#/sec] (mean)
```

![media rendimiento memcached](https://i.imgur.com/A6F05MM.png)

### 2.6

> ¿Se ha aumentado el rendimiento de forma significativa?

Bastante. Como he mostrado, hemos llegado a las mil peticiones por segundo.

## Varnish

### 3.1

> Reutilizar el escenario anterior

Borro la máquina y la creo de nuevo:

```console
vagrant destroy
vagrant up
```

Entro en ella y apunto su IP: 192.168.121.53

Modifico `ansible/hosts` con esta IP:

```console
[servidores_web]
servidor_web ansible_ssh_host=192.168.121.53 ansible_ssh_user=vagrant ansible_ssh_private_key_file=../.vagrant/machines/default/libvirt/private_key ansible_python_interpreter=/usr/bin/python3
```

Ejecuto el playbook para configurar la máquina:

```console
ansible-playbook site.yaml
```

### 3.2

> Configurar la resolución estática

`/etc/hosts`:

```console
# Varnish
192.168.121.53 www.example.org
```

### 3.3

> Instalar Varnish

```console
sudo apt install varnish
```

### 3.4

> Cambiar Nginx al puerto 8080

`/etc/nginx/sites-available/default`:

```console
server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
```

Reinicio:

```console
sudo systemctl restart nginx
```

Compruebo que ha cambiado:

![nginx 8080](https://i.imgur.com/wd7ge5y.png)

### 3.5

> Cambiar Varnish al puerto 80

Según la [documentación oficial de varnish](https://varnish-cache.org/docs/trunk/tutorial/putting_varnish_on_port_80.html) no hace falta modificar el fichero `/etc/default/varnish` en versiones recientes de Debian *(ya que es legacy)*.

En su lugar, modifico `/lib/systemd/system/varnish.service`:

![varnish p80](https://i.imgur.com/cvADw3h.png)

Reinicio la configuración de los daemon:

```console
sudo systemctl daemon-reload
```

Reinicio Varnish:

```console
sudo systemctl restart varnish
```

Compruebo que ha cambiado el puerto:

![varnish 80](https://i.imgur.com/lNGMvcj.png)

### 3.6

> Comprobar que la sección backend en `/etc/varnish/default.vcl` que tenemos por defecto sea correcta

Lo es:

![varnish backend](https://i.imgur.com/iRv235p.png)

### 3.7

> Comprobar que se puede acceder a Wordpress normalmente tras los cambios

![wordpress tras varnish](https://i.imgur.com/zzfw0Y6.png)

### 3.8

> Realizar las pruebas de rendimiento

```console
Requests per second:    5806.68 [#/sec] (mean)
Requests per second:    5952.47 [#/sec] (mean)
Requests per second:    5941.95 [#/sec] (mean)
Requests per second:    5877.94 [#/sec] (mean)
Requests per second:    5960.23 [#/sec] (mean)
```

![media rendimiento varnish](https://i.imgur.com/zPjEbrz.png)

Casi llegamos a una media de 6.000 peticiones por segundo, así que hemos aumentado el rendimiento considerablemente.

### 3.9

> ¿Cuántas peticiones llegan realmente a Nginx?

`/var/log/nginx/access.log`:

![peticiones reales en nginx](https://i.imgur.com/Ag5hXYm.png)

Como vemos, fijándonos en la hora, Nginx sólo ha servido 2 peticiones de las miles que se han hecho.
