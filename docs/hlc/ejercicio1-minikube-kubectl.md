# Ejercicio 1: Instalación y configuración de minikube y kubectl

[Documentación del curso](https://github.com/josedom24/curso_kubernetes_ies)

## Parte 1

Descargo el binario:

```shell
sudo apt update
sudo apt install curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Instalo el binario en una ruta disponible en el PATH:

```shell
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Compruebo que funciona:

```shell
vagrant@k1:~$ minikube version
minikube version: v1.29.0
commit: ddac20b4b34a9c8c857fc602203b6ba2679794d3
```

## Parte 2

Necesito KVM para usar minikube y que se pueda crear y controlar la VM con Kubernetes:

```shell
sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst libvirt-daemon
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```

## Parte 3

Lanzo minikube usando KVM:

```shell
minikube start --driver=kvm2
```

Puedo ver el estado:

```shell
vagrant@k1:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Parte 4

Instalo el cliente de Kubernetes con snap, para tener la versión más actualizada:

```shell
sudo apt install snapd
sudo snap install kubectl --classic
```

Compruebo que funciona, pero aún no está conectado con minikube:

```shell
vagrant@k1:~$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-19T02:26:55Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Unable to connect to the server: dial tcp 192.168.39.129:8443: connect: no route to host
```

Reinicio minikube para que reconfigure kubectl:

```shell
vagrant@k1:~$ minikube stop
✋  Stopping node "minikube"  ...
🛑  1 node stopped.
vagrant@k1:~$ minikube start
😄  minikube v1.29.0 on Debian 11.6 (kvm/amd64)
✨  Using the kvm2 driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🔄  Restarting existing kvm2 VM for "minikube" ...
🐳  Preparing Kubernetes v1.26.1 on Docker 20.10.23 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Compruebo que ya sí funciona el cliente completamente, y que está conectado con minikube:

```shell
vagrant@k1:~$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-19T02:26:55Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:51:25Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
```

Podemos ver los nodos de Kubernetes:

```shell
vagrant@k1:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   57m   v1.26.1
```

Para activar el autocompletado:

```shell
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```

## Comprobaciones finales

```shell
vagrant@k1:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

```shell
vagrant@k1:~$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    control-plane   13h   v1.26.1   192.168.39.129   <none>        Buildroot 2021.02.12   5.10.57          docker://20.10.23
```
