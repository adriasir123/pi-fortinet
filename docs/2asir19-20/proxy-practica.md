---
title: "Práctica: Proxy, proxy inverso y balanceadores de carga"
---

# Parte 1: Proxy

## Tarea 1: Instala squid en la máquina `proxy` y configúralo para que permita conexiones desde la red donde esté tu ordenador

```
sudo apt update
sudo apt install squid
```

Para permitir las conexiones desde la red del PC (la host-only que hay creada para la comunicación en específico)

acl mynetwork src 192.168.200.0/255.255.255.0
http_access allow mynetwork
http_access deny all












# Parte 2: Balanceador de carga






# Parte 3: Proxy inverso


