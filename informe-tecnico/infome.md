# Informe Técnico

## Índice

- [Introducción](#introducción)
- [Para desarrollo](#pasos-iniciales)
  - [Instalaciones necesarias](#instalaciones-necesarias)
  - [Clonar repositorios](#clonar-repositorios)
  - [Construir componentes](#construir-componentes)
  - [Obtener builds](#obtener-builds)
  - [Ejecutar componentes individualmente](#ejecutar-componentes-individualmente)
  - [Entorno de docker](#entorno-de-docker)
  - [Comandos Utiles](#comandos-utiles)

- [Kubernete Enviroments](#kubernetes-enviroment)
  - [Kubernetes OnRamp](#kubernetes-onramp)
  - [Kubernetes para desarrollo](#kubernetes-para-desarrollo)
    - [Requisitos](#requisitos)
    - [Intalación-del-entorno-de-kubernetes](#intalación-del-entorno-de-kubernetes)
    - [Trabajando con Helm](#trabajando-con-helm)
      - [Instalación de Helm](#instalando-helm)
      - [Obtener y operar Helm Charts](#obtener-y-operar-charts)
    - [Comandos para eliminar kubernetes](#comandos-para-eliminar-kubernetes)

## Introducción

Este informe técnico, recopila todo lo realizado para tener un entorno de desarrollo, el cual nos permita integrar nuevos cambios a los elementos presentados por Aether.

Para trabajar segun nuestras necesidades hicimos forks de algunos de los repos de Aether, los cuales se pueden encontrar en los siguientes enlaces:

GitHub repository for the OMEC Project ([https://github.com/omec-project](https://github.com/omec-project)): Microservices for SD-Core, plus the emulator (gNBsim) that subjects SD-Core to RAN workloads.

GitHub repository for the ONOS Project ([https://github.com/onosproject](https://github.com/onosproject)): Microservices for SD-RAN and ROC, plus the YANG models used to generate the Aether API.

GitHub repository for the ONF: [https://github.com/opennetworkinglab](https://github.com/opennetworkinglab) — OnRamp documentation and playbooks for deploying Aether.

Los forks los puede encontrar aquí en [este enlace](https://github.com/orgs/networkgcorefullcode/repositories).

En nuestro caso editamos el CI para adecuarlo a nuestras necesidades.

Al hacer forks podemos contribuir en un futuro al proyecto.

Las imagenes de docker se guardan en docker hub, si buscas network5gcore en docker hub deben de salir las imagenes.

---

## Pasos iniciales

Objetivo: construir las imagenes de cada uno de los componentes de Aether, utilizando nuestra configuracion hacer un entorno valido para el desarrollo. Hacer ese entorno utilizando solo docker.

### Instalaciones necesarias

Las instalaciones necesarias son las siguientes:

- [Go](https://go.dev/doc/install)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download)
- Python version >= 3.8

### Clonar repositorios

En nuestro entorno ejecutar los siguientes comandos:

```bash
cd ~
mkdir aether-forks
cd aether-forks
```

Crear aqui el archivo `python_get_repos.py`:

```bash
touch python_get_repos.py
```

Copiar el siguiente código (Para clonar todos los repo rapidamente):

```python
import requests
import os

user = "networkgcorefullcode"  # Reemplaza con el nombre de usuario
url = f"https://api.github.com/users/{user}/repos?per_page=100"
repos = requests.get(url).json()

for repo in repos:
    os.system(f"git clone {repo['clone_url']}")
```

```bash
python3 python_get_repos.py
```

Después de que termine la ejecución del script tendremos las siguientes carpetas:

![Estructura de carpetas después de clonar los repositorios](imgs/{3A4EB7A6-8BC8-4E09-89EB-5599B0EB2BB5}.png)

Actualmente al clonar los repositorios se clonara el repo utilFiles, en el cual se encuentran definidos varios de los archivos que definimos aquí, para poder utilizar estos archivos deberemos copiar su contenido en la raíz donde se encuentran todos los demas repos. Así los podremos utilizar sin problemas

### Construir componentes

Crear el script `build_components.sh`:

```bash
touch build_components.sh
sudo chmod 700 build_components.sh
```

```bash
#!/bin/bash

for dir in "$PWD"/*/; do
    [ -d "$dir" ] && echo "Directorio: $dir"
    if (cd "$dir" && make all); then
        :
    else
        echo "error al ejecutar make all"
    fi
done
```

Ejecutar el script para crear los builds de cada componente:

```bash
./build_components.sh
```

### Obtener builds

Crear el script `get_builds.sh`:

```bash
touch get_builds.sh
sudo chmod 700 get_builds.sh
```

```bash
#!/bin/bash

# Guardar el directorio actual en una variable
current_dir="$PWD"

# Recorre todos los directorios en la raíz del script
for dir in "$current_dir"/* ; do
    echo "Revisando $dir"
    # Verifica si existe la carpeta bin dentro del directorio
    if [ -d "$dir/bin" ]; then
        # Copia el contenido de bin al directorio actual
        cp -r "$dir/bin/" "$current_dir"
        echo "Contenido de $dir/bin copiado a $current_dir"
    fi
done
```

Este script copiara todos los builds de go en una carpeta llamada bin

```bash
./get_builds.sh
```

![Binario de cada uno de los componentes](imgs/{BBEA4130-7110-4E58-8C59-AC1895312AAC}.png)

Binario de cada uno de los componentes

## Ejecutar componentes individualmente

Para ejecutar los componentes individualmente y hacer pruebas en cada uno de ellos, podemos hacer lo siguiente:

1. Asegurarnos de que el componente que queremos ejecutar tenga su binario en la carpeta `bin`.
2. Abrir una terminal y navegar a la carpeta `bin` donde se encuentran los binarios de los componentes.
3. Ejecutar el binario del componente deseado. Por ejemplo, si queremos ejecutar el componente `amf`, podemos usar el siguiente comando:

```bash
cd ~/aether-forks/bin
```

```bash
./amf --cfg ~/aether-forks/configs_files/amfcfg.yaml
```

Asi para cada uno de los componentes, que soporten una configuracion inicial a través de un archivo YAML de configuración.

Esto nos permitirá probar cada componente de forma individual y verificar su funcionamiento antes de integrarlos en un entorno más complejo como Docker o Kubernetes. Es espcialmente útil para el desarrollo y la depuración de cada componente por separado.

## Entorno de docker

Para crear un entorno de desarrollo utilizando Docker, podemos utilizar un archivo `docker-compose.yaml` que defina los servicios necesarios para ejecutar los componentes de Aether. A continuación se muestra un ejemplo básico de cómo podría ser este archivo:

En la carpeta `configs_files/` se deben colocar los archivos de configuración YAML para cada componente, como `amfcfg.yaml`, `ausfcfg.yaml`, etc. Estos archivos deben contener la configuración específica para cada componente.

En el repo `utilFiles` actualmente hay una serie de docker-compose y script que levantan un entorno de docker, segun las configuraciones asociadas en `configs_files`

Hasta ahora todo es una prueba, la configuracion puede que no sea estable, cualquier correcion de la misma será bienvenida. La idea es tener el entorno de prueba sin problemas, que sea facil desarrollar y comprobar los resultados en nuestro entorno.

## Comandos Utiles

- Para detener todos los contenedores:

```bash
docker compose down
```

- Para ver los logs de un contenedor específico:

```bash
docker compose logs <nombre_del_contenedor>
```

- Para ejecutar comandos dentro de un contenedor en ejecución:

```bash
docker exec -it <nombre_del_contenedor> <comando>
```

Por ejemplo, para abrir una terminal bash en el contenedor `amf`:

```bash
docker exec -it amf /bin/bash
```

- Para reconstruir los contenedores después de hacer cambios en el código:

```bash
docker compose up -d --build
```

Si necesitas instalar utilidades adicionales como `vim`, `strace`, `net-tools`, `curl`, `netcat-openbsd` y `bind-tools` en un contenedor basado en Alpine Linux, puedes ejecutar el siguiente comando dentro del contenedor:

```bash
apk update && apk add --no-cache -U vim strace net-tools curl netcat-openbsd bind-tools
```

Esto actualizará los índices de paquetes e instalará las herramientas necesarias sin guardar archivos temporales, manteniendo la imagen ligera.

Script para instalar herramientas en los contenedores core 5G:

```bash
#!/bin/bash

# Lista de contenedores core 5G según docker-compose
core5g_containers=(amf ausf nrf nssf pcf smf udm udr)

for c in "${core5g_containers[@]}"; do
    echo "Instalando herramientas en $c..."
    docker exec -it "$c" sh -c "apk update && apk add --no-cache -U vim strace net-tools curl netcat-openbsd bind-tools"
done
```

## Kubernetes Enviroment

En las siguientes secciones se abordaran configuraciones relacionadas a un entorno de kubernetes, ambiente donde Aether fue diseñado para desplegarse

## Kubernetes OnRamp

## Kubernetes para desarrollo

### Requisitos

- Ubuntu Server version 22.04 or later
- Docker
- Kubernetes
- Go version 1.24.4 or later

---

### Intalación del entorno de Kubernetes

---

Para desplegar Aether debemos tener un entorno de kubernetes, en el cual utilizando los diferentes charts de helm desplegaremos los diferentes servicios

#### ✅ 1. **Preparar el Servidor**

Asegúrate de que tu Server tenga al menos:

- ✅ 2 CPUs (4 si vas a desplegar Aether)
- ✅ 4–8 GB de RAM (mejor con 8 GB para SD-Core)
- ✅ 20+ GB de almacenamiento
- ✅ Sistema operativo Ubuntu Server 22.04 LTS
- ✅ Seguridad: grupo de seguridad con puertos abiertos (SSH, 6443, 80, 443, 22, etc.)

---

#### ✅ 2. **Configurar el entorno base**

```bash
# Actualizar paquetes
sudo apt update && sudo apt upgrade -y

# Desactivar swap (requisito de kubeadm)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

#### ✅ 3. **Instalar Docker (container runtime)**

```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Habilitar Docker
sudo systemctl enable docker
sudo systemctl start docker
```

---

#### ✅ 4. **Instalar Kubernetes (kubeadm, kubelet, kubectl)**

```bash
sudo apt-get update -y
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```

---

#### ✅ 5. **Inicializar el clúster (modo single-node para pruebas)**

```bash
# (opcional) Usa tu IP pública o privada como advertise address
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}') \
  --cri-socket=/var/run/containerd/containerd.sock

```

> ✅ Esto instalará un clúster con Calico/Flannel-compatible pod CIDR.

Si presentar problemas relacionados con que no encuentra el containerd.sock, puedes hacer lo siguiente:

Asegúrate de que containerd esté instalado

```bash
which containerd
```

Debe devolver algo como:

```bash
/usr/bin/containerd
```

Asegúrate de que containerd esté corriendo

```bash
sudo systemctl status containerd
```

Si no está corriendo, intenta:

```bash
sudo systemctl start containerd
```

Verifica que el archivo de configuración de containerd tenga habilitado el CRI

Ejecuta:

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```

Luego edita el archivo:

```bash
sudo nano /etc/containerd/config.toml
```

Busca esta sección:

```toml
[plugins."io.containerd.grpc.v1.cri"]
```

> Asegúrate de que no esté comentada y que esté habilitada. También asegúrate de que tenga:

```toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

Reinicia containerd después de los cambios

```bash
sudo systemctl restart containerd
```

Verifica el socket
Asegúrate de que el socket exista:

```bash
ls -l /var/run/containerd/containerd.sock
```

> Debe aparecer como archivo tipo socket.

Vuelve a intentar la inicialización

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}') \
  --cri-socket=/var/run/containerd/containerd.sock
```

---

#### ✅ 6. **Configurar el entorno para el usuario actual**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

#### ✅ 7. **Instalar red de pods (ej. Calico)**

```bash
# Usando Calico como red de pods
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

---

#### ✅ 8. **Permitir que el nodo actúe como master y worker (modo prueba)**

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

Esto es necesario si solo tienes **una máquina** y quieres que los pods de usuario (como Aether) se ejecuten ahí.

---

#### ✅ 9. **Verifica que todo está funcionando**

```bash
kubectl get nodes
kubectl get pods -A
```

Debes ver algo como esto:

![alt text](imgs/{70E4BAB7-F16D-481C-AF22-A3AF4EC88405}.png)

### Trabajando con Helm

#### Instalando Helm

Ejecuta:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

#### Obtener y operar charts

Aether proporciona una serie de charts de helm, los cuales podemos configurar segun nuestras necesidades. Estos charts los podemos encontrar en:

[https://charts.aetherproject.org](https://charts.aetherproject.org)

[https://charts.onosproject.org](https://charts.onosproject.org)

[https://charts.opencord.org](https://charts.opencord.org)

[https://charts.atomix.io](https://charts.atomix.io)

[https://sdrancharts.onosproject.org](https://sdrancharts.onosproject.org)

[https://charts.rancher.io/](https://charts.rancher.io/)

Los repos de github son los siguientes:

ROC: [https://github.com/onosproject/roc-helm-charts](https://github.com/onosproject/roc-helm-charts)

SD-RAN: [https://github.com/onosproject/sdran-helm-charts](https://github.com/onosproject/sdran-helm-charts)

SD-Core: [https://github.com/omec-project/sdcore-helm-charts](https://github.com/omec-project/sdcore-helm-charts).

De estos repos hicimos repositorio para trabajar segun nuestras necesidades.

SD-Core: [https://github.com/networkgcorefullcode/helm-charts](https://github.com/networkgcorefullcode/sdcore-helm-charts).

Ejecutar los siguientes comandos:

Opcional
```bash
helm repo add stable https://charts.helm.sh/stable                        
helm repo add cord https://charts.opencord.org                          
helm repo add atomix https://charts.atomix.io                             
helm repo add onosproject https://charts.onosproject.org                       
helm repo add sdran https://sdrancharts.onosproject.org                  
helm repo add aether https://charts.aetherproject.org                     
helm repo add cetic https://cetic.github.io/helm-charts                  
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo onos

## para revertir esto
helm repo remove stable
helm repo remove cord
helm repo remove atomix
helm repo remove onosproject
helm repo remove sdran
helm repo remove aether
helm repo remove cetic
helm repo remove bitnami
```

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Ejecutar los siguientes comandos desde el directorio `helm-charts/`:

```bash
helm dependency build atomix-1.1.2/chart
helm dependency build onos-operator
cd aether-roc-umbrella
rm Chart.lock
helm dependency build .
cd ..
cd sd-core
rm Chart.lock
helm dependency build .
```

Ejecutar ahora:

```bash
helm install -n kube-system atomix atomix-1.1.2/chart
helm install -n kube-system onos-operator onos-operator
kubectl create namespace roc5g
kubectl create namespace sdcore
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset.yml
kubectl get crd network-attachment-definitions.k8s.cni.cncf.io
helm install -n sdcore core5g sd-core
helm install -n roc5g rocamp aether-roc-umbrella

# para revertir lo anterior

helm uninstall -n sdcore core5g
helm uninstall -n roc5g rocamp
```

Revisar los logs y verificar que todo funcione correctamente, si hay errores en el despliegue, se debe revisar la configuracion proporcionada en el helm chart correspondiente.

Puertos de los servicios

| Servicio           | Puerto | Descripción                |
|--------------------|--------|----------------------------|
| amf                | 38412  | Interfaz NGAP              |
|                    |        |                            |
| smf                | 8805   | Interfaz PFCP              |
| ausf               | 8080   | API REST                   |
| nrf                | 8000   | Registro de funciones      |
| pcf                | 8081   | Políticas de control       |
| udm                | 8082   | Gestión de datos de usuario|
| udr                | 8083   | Repositorio de datos       |
| webui              | 31002  | Interfaz para el Webui     |

### Comandos para eliminar kubernetes

```bash
sudo systemctl stop kubelet
sudo systemctl stop docker

sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
sudo apt-get autoremove

# Reiniciar la pc en este paso
sudo rm -rf ~/.kube
sudo rm -rf /etc/kubernetes/
sudo rm -rf /var/lib/etcd/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /var/lib/dockershim/
sudo rm -rf /var/run/kubernetes/
sudo rm -rf /etc/cni/
sudo rm -rf /opt/cni/
sudo rm -rf /opt/local-path-provisioner

sudo docker system prune -a --volumes

kubectl version
# Debe decir comando no encontrado

kubeadm version
# Debe decir comando no encontrado

sudo rm -rf $HOME/.cache/helm
sudo rm -rf $HOME/.config/helm
sudo rm -rf $HOME/.local/share/helm
```
