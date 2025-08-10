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
  - [Aether OnRamp](#aether-onramp)
  - [Kubernetes para desarrollo](#kubernetes-para-desarrollo)
    - [Requisitos](#requisitos)
    - [Intalación-del-entorno-de-kubernetes](#intalación-del-entorno-de-kubernetes)
    - [Trabajando con Helm](#trabajando-con-helm)
      - [Instalación de Helm](#instalando-helm)
      - [Obtener y operar Helm Charts](#obtener-y-operar-charts)
      - [Utils](#utils)
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

## Aether OnRamp

### Imágenes de Docker

Para desplegar los componentes de Aether actualizados, es necesario tener las imágenes de cada NF. Para ello se realizaron los siguientes pasos.

1. En el repo de cada NF se editó el archivo `VERSION` y se cambió a un valor personalizado, en este caso fue: `v1.2.1-new-dev`

2. En el archivo `Makefile` se completaron las siguientes variables:
```makefile
DOCKER_REGISTRY ?= 192.168.12.15:8083/
DOCKER_REPOSITORY ?= omecproject/
```
- `DOCKER_REGISTRY` se configuró con el valor del registry privado de Docker que se encuentra desplegado en un servidor Nexus en los servidores de la empresa
- `DOCKER_REPOSITORY` se configuró con nombre del repositorio por defecto de Aether SD-Core

3.  Hacer un `docker build` para construir las imágenes y luego un `docker push` para subirlas al rregistry. Para hacer el *push* es necesario primero loguearse en el Nexus con `docker login 192.168.12.15:8083`.

Todos estos pasos pueden adaptarse según la conveniencia del usuario, usar una version diferente o subir las imagenes a otro registry de Docker.


### Configuración del *values* de Helm

El archivo de *values* de Helm se encuentra en la siguiente ruta, partiendo desde el directorio raíz del repositorio de Aether OnRamp: `/deps/5gc/roles/core/templates/sdcore-5g-values.yaml`

En este archivo se hicieron varias configuraciones.

1. Se configuró el despliegue para que bajara las imágenes actualizadas del registry privado en Nexus de la siguiente forma:

```yaml
5g-control-plane:
  enable5G: true
  images:
    repository: "192.168.12.15:8083/"
    tags:
      amf: omecproject/5gc-amf:v1.2.1-new-dev
      ausf: omecproject/5gc-ausf:v1.2.1-new-dev
      smf: omecproject/5gc-smf:v1.2.1-new-dev
      udm: omecproject/5gc-udm:v1.2.1-new-dev
      udr: omecproject/5gc-udr:v1.2.1-new-dev
      pcf: omecproject/5gc-pcf:v1.2.1-new-dev
      nrf: omecproject/5gc-nrf:v1.2.1-new-dev
      webui: omecproject/5gc-webui:v1.2.1-new-dev
      sctplb: omecproject/sctplb:v1.2.1-new-dev
      nssf: omecproject/5gc-nssf:v1.2.1-new-dev
      upfadapter: omecproject/upfadapter:v1.2.2-new-dev


# otras configuraciones....

omec-sub-provision:
  enable: true
  images:
    repository: "192.168.12.15:8083/"
    tags:
      simapp: omecproject/simapp:v1.2.1-new-dev


# otras configuraciones....

omec-user-plane:
  enable: true
  nodeSelectors:
    enabled: true
  resources:
    enabled: false
  images:
    repository: "192.168.12.15:8082/"
     #tags:
       #bess: omecproject/upf-epc-bess:v1.2.1-new-dev
       #pfcpiface: omecproject/upf-epc-pfcpiface:v1.2.1-new-dev
      # tools: omecproject/busybox:stable

# otras configuraciones....

```
- Es importante destacar que el `tag` definido para cada imagen debe contener la siguiente estructura debido a que así es como se estructura la imagen en el registry
```
omecproject/5gc-<nombre del componente en minúscula>:<valor definido en el archivo VERSION del repo del componente>
```
- Por ahora los despliegues se han hecho manteniendo el plano de usuario original de Aether, como se puede observar las imágenes actualizadas de esa sección están definidas pero comentadas.

2. Se añadieron configuraciones para varias NFs (AUSF, UDM, UDR) y fueron modificadas otras (WebUI, AMF, NRF, entre otras). En este documento no se detallará cada configuración debido a que sería demasiado extenso. Para una inspección completa puede acceder al archivo [aquí](https://gitlab.generalsoftwareinc.com/5g/aether/-/blob/feature/update-aether/deps/5gc/roles/core/templates/sdcore-5g-values.yaml?ref_type=heads). 

3. Debido a que se están usando componentes de Aether más actualizados (no solo por este proyecto sino también por desarrolladores oficiales de Aether) existen procesos nuevos. Uno de ellos es que ahora las NFs hacen un *polling* periódico al **WebUI** por el puerto `5001`. Es por eso que cada NF debe tener esta configuración.
```yaml
 webuiUri: "http://webui:5001"
```
El manifiesto del ***service*** del **WebUI** no tiene este puerto configurado, por lo que se hizo necesario aplicar una solución que permitiera exponer el puerto y hacer permanente este cambio. Para ello se utilizó [Kustomize](https://kustomize.io) que permite aplicar modificaciones a manifiestos de Kubernetes. Kustomize tiene definido en un archivo la nueva configuración para el *service* del WebUI y la aplica como un parche al manifiesto final que se renderiza con Helm. Este proceso esta incluido en el despliegue del 5GC con Ansible.

### Simulación

Por el momento todos los Pods de Aether llegan al estado `Running` y se han realizado simulaciones para comprobar un funcionamiento correcto. Se ha obtenido como resultado que la `gnb` simulada se conecta exitosamente al 5GC pero a la hora de conectar el `ue` ocurren fallos. Los errores están en proceso de debugueo.

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

Opcionales:

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
kubectl label node <node_name> node-role.aetherproject.org=omec-upf
helm install -n sdcore core5g sd-core
helm install -n roc5g rocamp aether-roc-umbrella

# para revertir lo anterior

helm uninstall -n sdcore core5g
helm uninstall -n roc5g rocamp
```

#### Utils

Para habilitar una instancia de mongodb express que nos provea de una interfaz web para visualizar facilmente la base de datos mongodb. Crear y guardar esto en un nuestro entorno

```yml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app.kubernetes.io/name: mongodb
  ports:
    - port: 27017
      targetPort: 27017
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
        - name: mongo-express
          image: mongo-express:latest
          ports:
            - containerPort: 8081
          env:
            - name: ME_CONFIG_MONGODB_SERVER
              value: "mongodb"  # Nombre del servicio de MongoDB
            - name: ME_CONFIG_MONGODB_PORT
              value: "27017"
            - name: ME_CONFIG_BASICAUTH_USERNAME
              value: "admin"
            - name: ME_CONFIG_BASICAUTH_PASSWORD
              value: "admin123"
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express
spec:
  type: NodePort
  selector:
    app: mongo-express
  ports:
    - port: 8081
      targetPort: 8081
      nodePort: 30081
```

Ejecutar el comando apply correspondiente

```bash
kubectl apply -f <file_name> -n <name_space>
```

Revisar los logs y verificar que todo funcione correctamente, si hay errores en el despliegue, se debe revisar la configuracion proporcionada en el helm chart correspondiente.

Puertos de los servicios

| Servicio           | Puerto | Descripción                               |
|--------------------|--------|-------------------------------------------|
| webui              | 30001  | Interfaz para el Webui                    |
| mongodb-express    | 30081  | Interfaz web para interactuar con mongodb |

Para ejecutar el user-plane en una instancia EC2 de AWS, seguir los siguientes pasos

1. **Requisitos de red (VPC prerequisites)**

   * Crear una VPC con tres subredes: `mgmt`, `access` y `core`.
   * Hacer accesibles a internet las subredes `mgmt` y `core`.
   * Configurar rutas (incluyendo para eNB/gNB si están en on-premise).
   * Crear interfaces de red (ENIs) para `access` y `core`, desactivar el *source/destination check* en `core` y añadir ruta para la subred de UEs.

2. **Lanzar la instancia EC2**

   * Usar un tipo con **ENA** habilitado (ej. `c5.2xlarge`) y AMI Ubuntu 20.04.
   * Inicialmente conectar solo la subred `mgmt`, y luego añadir las ENIs de `access` y `core`.

3. **Configurar el sistema operativo**

   * Habilitar **HugePages de 1Gi** modificando GRUB.
   * Cargar y configurar el driver **vfio-pci** con modo *unsafe IOMMU*.
   * Asociar las interfaces `access` y `core` al driver vfio-pci usando `driverctl`.

4. **Configurar Kubernetes (K8S)**

   * Instalar **Multus CNI**.
   * Instalar y configurar el **SR-IOV device plugin** para las interfaces de `access` y `core` (asignadas por PCI).
   * Verificar recursos disponibles: HugePages y dispositivos SR-IOV.
   * Instalar el binario `vfioveth` en `/opt/cni/bin`.

5. **Instalar BESS-UPF**

   * Requiere helm chart con licencia de miembros ONF.
   * Usar un `overriding-values.yaml` con configuración de subredes, direcciones IP, MAC y recursos SR-IOV.
   * Desplegar con `helm install` y verificar que el pod `upf-0` esté en estado *Running*.


#### Preparación para un entorno EC2 AWS

Preparación del VPC

---

1️⃣ Crear la **VPC**

1. Ve a **VPC → Your VPCs → Create VPC**.
2. Modo: **VPC only**.
3. Nombre: `upf-vpc`.
4. IPv4 CIDR: por ejemplo `10.0.0.0/16`.
5. Crear.

---

2️⃣ Crear las **subredes**

Necesitamos **tres subnets** dentro de la misma VPC:

* **mgmt** → para administrar la instancia (acceso SSH, K8s API, etc.)
* **access** → tráfico hacia/desde gNB/eNB
* **core** → tráfico hacia el core de red (internet o N6)

Ejemplo:

| Nombre | CIDR        | Zona AZ    |
| ------ | ----------- | ---------- |
| mgmt   | 10.0.1.0/24 | us-east-1a |
| access | 10.0.2.0/24 | us-east-1a |
| core   | 10.0.3.0/24 | us-east-1a |

En la consola:

1. **VPC → Subnets → Create subnet**
2. Selecciona `upf-vpc` y crea las 3 subnets con sus CIDRs.

---

3️⃣ Hacer accesibles a Internet las subredes **mgmt** y **core**

Hay varias formas:

* Opción fácil: asignarlas a un **Internet Gateway**.
* Opción del ejemplo: usar **NAT Gateway** en una subred pública.

Ejemplo con NAT Gateway:

1. **VPC → Internet Gateways → Create internet gateway** → Adjuntar a `upf-vpc`.
2. Crea una **subred pública** extra, ej. `10.0.10.0/24`, con **Auto-assign Public IPv4** habilitado.
3. **Elastic IPs** → reservar una IP y usarla al crear el NAT Gateway.
4. **VPC → NAT Gateways → Create NAT Gateway** en la subred pública.
5. En las **Route Tables** de `mgmt` y `core`:

   * Añadir ruta `0.0.0.0/0` → hacia el **NAT Gateway** esto en las routes table de core
   * Añadir ruta `0.0.0.0/0` → hacia el **Internet Gateway** esto en las routes table de mgmt

---

4️⃣ Configurar rutas para **access**

Si tienes gNB/eNB **on-premise**:

* Ve a **Route Tables** de `access` y añade rutas hacia las redes de las estaciones base, apuntando al gateway correcto (VPN, Direct Connect, etc.).

---

5️⃣ Crear **ENIs** para access y core

1. **EC2 → Network Interfaces → Create network interface**.

   * Para `access`: subnet `access`, sin IP pública.
   * Para `core`: subnet `core`, sin IP pública.
2. Guarda los **IDs** de cada ENI (los usarás luego para adjuntarlos a la EC2).

---

6️⃣ Desactivar **source/destination check** en la ENI de core

Esto permite que la interfaz enrute tráfico que no sea solo para ella.

1. Selecciona la ENI de `core`.
2. **Actions → Change source/dest. check → Disable**.

---

7️⃣ Añadir ruta para la **UE pool subnet**

Esto es para que el tráfico hacia los UEs pase por la interfaz core.

1. Ve a la **Route Table** de la subred pública o la que use el internet/NAT.
2. Añade ruta:

   * Destination: `SUBRED_UE_POOL` (ej. `10.22.0.128/26`)
   * Target: **ENI core**.

---

Preparación de la instancia EC2

---

**1️⃣ Lanzar la instancia EC2**

1. Ve a **AWS Console → EC2 → Launch instance**.
2. **Name**: por ejemplo `ec2-server`.
3. **AMI**: selecciona **Ubuntu Server 22.04 LTS (64-bit x86)**.
4. **Instance type**: selecciona **c5.2xlarge** (o cualquier tipo con ENA habilitado y suficiente rendimiento).
5. **Key pair**: selecciona o crea uno para poder conectarte por SSH.

---

**2️⃣ Configurar red inicial (solo `mgmt`)**

1. En **Network settings**:

   * **VPC**: selecciona la que creaste (`upf-vpc` o el nombre que tenga).
   * **Subnet**: elige **solo la subred mgmt** (la que tiene acceso a Internet o NAT Gateway).
   * **Auto-assign Public IP**: habilitado, para que puedas conectarte por SSH.
2. En **Security group**:

   * Asegúrate de permitir **SSH (TCP 22)** desde tu IP.

---

**3️⃣ Lanzar y conectarte**

* Haz clic en **Launch instance**.
* Conéctate por SSH a la IP pública:

```bash
ssh -i tu_clave.pem ubuntu@IP_PUBLICA
```

---

**4️⃣ Crear y adjuntar las ENIs de `access` y `core`**

**(Esto se hace después de que la instancia está corriendo)**

1. Ve a **EC2 → Network Interfaces → Create network interface**. Esto se indico en los pasos anteriores

   * **Name**: `eni-access`.
   * **Subnet**: subred `access`.
   * **Auto-assign Public IP**: desactivado (solo se necesita en `mgmt`).
   * Crea otra igual pero para `core` (`eni-core`).

2. **Desactivar Source/Destination Check** en la ENI `core`:

   * Selecciona la ENI `eni-core`.
   * En **Actions → Change source/dest. check → Disable**.

3. **Adjuntar las ENIs a la instancia**:

   * En la ENI `eni-access`, ve a **Actions → Attach** y elige tu instancia.
   * Repite para `eni-core`.

---

**5️⃣ Verificar desde el SO**

Dentro de la instancia, ejecuta:

```bash
ip addr
```

Deberías ver:

* `ens5` → interfaz de `mgmt` (con IP pública o detrás de NAT).
* Otra interfaz para `access`.
* Otra interfaz para `core`.

---

Configuracion del sistema

SSH a la máquina virtual y actualiza Grub para habilitar HugePages de 1Gi.

```bash
$ sudo vi /etc/default/grub
# Edit grub command line
GRUB_CMDLINE_LINUX="transparent_hugepage=never default_hugepagesz=1G hugepagesz=1G hugepages=2"

$ sudo update-grub
```

Carga el controlador vfio-pci y habilita el modo unsafe IOMMU.

``` bash
sudo su -
modprobe vfio-pci
echo "options vfio enable_unsafe_noiommu_mode=1" > /etc/modprobe.d/vfio-noiommu.conf
echo "vfio-pci" > /etc/modules-load.d/vfio-pci.conf
reboot
```

Después de reiniciar, verifica los cambios.

```bash
$ cat /proc/meminfo | grep Huge
# AnonHugePages:         0 kB
# ShmemHugePages:        0 kB
# FileHugePages:         0 kB
# HugePages_Total:       2
# HugePages_Free:        2
# HugePages_Rsvd:        0
# HugePages_Surp:        0
# Hugepagesize:    1048576 kB
# Hugetlb:         4194304 kB

$ cat /sys/module/vfio/parameters/enable_unsafe_noiommu_mode
# Y
```

Por último, asocia las interfaces `access` y `core` al controlador vfio.  
Antes de continuar, toma nota de las direcciones MAC de las dos ENIs, ya sea desde el panel de EC2 o usando el comando `aws ec2 describe-network-interfaces`.  
Estas direcciones MAC son necesarias para identificar la dirección PCI de cada interfaz.

```bash
# Find interface name of the access and core subnet ENIs by comparing the MAC address
$ ip link show ens6
$ ip link show ens7

# Find PCI address of the access and core subnet ENIs
$ lshw -c network -businfo
Bus info          Device           Class      Description
=========================================================
pci@0000:00:05.0  ens5             network    Elastic Network Adapter (ENA)
pci@0000:00:06.0  ens6             network    Elastic Network Adapter (ENA) # access in this example
pci@0000:00:07.0  ens7             network    Elastic Network Adapter (ENA) # core in this example

# Install driverctl
$ sudo apt update
$ sudo apt install driverctl

# Check current driver
$ sudo driverctl -v list-devices | grep -i net
0000:00:05.0 ena (Elastic Network Adapter (ENA))
0000:00:06.0 ena (Elastic Network Adapter (ENA))
0000:00:07.0 ena (Elastic Network Adapter (ENA))

# Bind access and core interfaces to vfio-pci driver
$ sudo driverctl set-override 0000:00:06.0 vfio-pci
$ sudo driverctl set-override 0000:00:07.0 vfio-pci

# Verify
$ sudo driverctl -v list-devices | grep -i net
# 0000:00:05.0 ena (Elastic Network Adapter (ENA))
# 0000:00:06.0 vfio-pci [*] (Elastic Network Adapter (ENA))
# 0000:00:07.0 vfio-pci [*] (Elastic Network Adapter (ENA))

$ ls -l /dev/vfio/
# crw------- 1 root root 242,   0 Aug 17 22:15 noiommu-0
# crw------- 1 root root 242,   1 Aug 17 22:16 noiommu-1
# crw-rw-rw- 1 root root  10, 196 Aug 17 21:51 vfio
```

Configurando k8s

La instalación de Kubernetes (K8S) está fuera del alcance de esta guía. Una vez que K8S esté listo en la máquina virtual, deberás instalar `Multus` y `sriov-device-plugin`.

```bash
# Install Multus
$ kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/release-3.7/images/multus-daemonset.yml

# Verify
$ kubectl get ds -n kube-system kube-multus-ds
# NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE
# kube-multus-ds   1         1         1       1            1

# Create sriov device plugin config
# Replace PCI address if necessary
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: sriovdp-config
data:
  config.json: |
    {
      "resourceList": [
        {
          "resourceName": "intel_sriov_vfio_access",
          "selectors": {
            "pciAddresses": ["0000:00:06.0"]
          }
        },
        {
          "resourceName": "intel_sriov_vfio_core",
          "selectors": {
            "pciAddresses": ["0000:00:07.0"]
          }
        }
      ]
    }
EOF

# Create sriov device plugin DaemonSet
$ kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/sriov-network-device-plugin/v3.3/deployments/k8s-v1.16/sriovdp-daemonset.yaml

# Verify
$ kubectl get ds -n kube-system kube-sriov-device-plugin-amd64
# NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE
# kube-sriov-device-plugin-amd64   1         1         1       1            1
```

Verifica los recursos asignables en el nodo. Asegúrate de que haya 2 HugePages de 1Gi, 1 `intel_sriov_vfio_access` y 1 `intel_sriov_vfio_core`.

```bash
$ kubectl get node  -o json | jq '.items[].status.allocatable'
{
  "attachable-volumes-aws-ebs": "25",
  "cpu": "7",
  "ephemeral-storage": "46779129369",
  "hugepages-1Gi": "2Gi",
  "hugepages-2Mi": "0",
  "intel.com/intel_sriov_vfio_access": "1",
  "intel.com/intel_sriov_vfio_core": "1",
  "memory": "13198484Ki",
  "pods": "110"
}
```

Por último, copia el binario `vfioveth` de CNI en la ruta `/opt/cni/bin` dentro de la máquina virtual.

```bash
sudo wget -O /opt/cni/bin/vfioveth https://raw.githubusercontent.com/opencord/omec-cni/master/vfioveth
sudo chmod +x /opt/cni/bin/vfioveth
```

Instalando bess upf con Helm

El chart de Helm de BESS UPF está actualmente bajo licencia exclusiva para miembros de ONF. Si ya tienes acceso al chart, proporciona los siguientes valores de override al desplegarlo. No olvides reemplazar correctamente las direcciones IP y MAC según tu entorno.

```bash
$ cat >> overriding-values.yaml << EOF
config:
  upf:
    privileged: true
    enb:
      subnet: "10.22.0.128/26"
    access:
      ip: "172.31.67.192/24"
      gateway: "172.31.67.1"
      mac: "02:ee:e9:c8:a5:31"
      resourceName: "intel.com/intel_sriov_vfio_access"
    core:
      ip: "172.31.68.136/24"
      gateway: "172.31.68.1"
      mac: "02:67:74:80:be:35"
      resourceName: "intel.com/intel_sriov_vfio_core"
EOF

$ helm install -n bess-upf bess-upf [path/to/helm/chart] -f overriding-values.yaml
$ kubectl get po -n bess-upf
# NAME    READY   STATUS    RESTARTS   AGE
# upf-0   4/4     Running   0          41h
```

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
