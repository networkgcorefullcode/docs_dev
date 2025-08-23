# Informe Técnico

## Índice

- [Informe Técnico](#informe-técnico)
  - [Índice](#índice)
  - [Introducción](#introducción)
  - [Configuración del entorno de desarrollo](#configuración-del-entorno-de-desarrollo)
    - [Pasos iniciales](#pasos-iniciales)
    - [Instalaciones necesarias](#instalaciones-necesarias)
    - [Clonar repositorios](#clonar-repositorios)
    - [Construir componentes](#construir-componentes)
    - [Obtener builds](#obtener-builds)
    - [Ejecutar componentes individualmente](#ejecutar-componentes-individualmente)
  - [Ejemplos de desarrollo](#ejemplos-de-desarrollo)
    - [NRF (NfProfile Update)](#nrf-nfprofile-update)
      - [Proceso de desarrollo](#proceso-de-desarrollo)
    - [WebUI Update](#webui-update)
    - [AMF, SMF, NSSF update to release 2.0.0](#amf-smf-nssf-update-to-release-200)
  - [Entorno de Docker](#entorno-de-docker)
  - [Comandos útiles](#comandos-útiles)
  - [Kubernetes Environment](#kubernetes-environment)
  - [Aether OnRamp](#aether-onramp)
    - [Imágenes de Docker](#imágenes-de-docker)
    - [Configuración del *values* de Helm](#configuración-del-values-de-helm)
  - [Kubernetes para desarrollo](#kubernetes-para-desarrollo)
    - [Requisitos](#requisitos)
    - [Instalación del entorno de Kubernetes](#instalación-del-entorno-de-kubernetes)
      - [1. **Preparar el servidor**](#1-preparar-el-servidor)
      - [2. **Configurar el entorno base**](#2-configurar-el-entorno-base)
      - [3. **Instalar Docker (container runtime)**](#3-instalar-docker-container-runtime)
      - [4. **Instalar Kubernetes (kubeadm, kubelet, kubectl)**](#4-instalar-kubernetes-kubeadm-kubelet-kubectl)
      - [5. **Inicializar el clúster (modo single-node para pruebas)**](#5-inicializar-el-clúster-modo-single-node-para-pruebas)
      - [6. **Configurar el entorno para el usuario actual**](#6-configurar-el-entorno-para-el-usuario-actual)
      - [7. **Instalar red de pods (ej. Calico)**](#7-instalar-red-de-pods-ej-calico)
      - [8. **Permitir que el nodo actúe como master y worker (modo prueba)**](#8-permitir-que-el-nodo-actúe-como-master-y-worker-modo-prueba)
      - [9. **Verifica que todo está funcionando**](#9-verifica-que-todo-está-funcionando)
    - [Trabajando con Helm](#trabajando-con-helm)
      - [Instalando Helm](#instalando-helm)
      - [Obtener y operar charts](#obtener-y-operar-charts)
      - [Utils](#utils)
    - [Comandos para eliminar kubernetes](#comandos-para-eliminar-kubernetes)
  - [Pruebas de simulación y trazas de logs de registro](#pruebas-de-simulación-y-trazas-de-logs-de-registro)
    - [Configuraciones previas](#configuraciones-previas)
      - [Despliegue de Aether  y UERANSIM](#despliegue-de-aether--y-ueransim)
      - [Simulación](#simulación)
      - [Cambio en la configuracion del *service* `nrf`](#cambio-en-la-configuracion-del-service-nrf)
      - [Instalación de Docker y Docker Compose](#instalación-de-docker-y-docker-compose)
      - [Despliegue del UDM de Open5GS](#despliegue-del-udm-de-open5gs)
      - [Evidencia de registro en los logs](#evidencia-de-registro-en-los-logs)
  - [Evaluación final y recomendaciones](#evaluación-final-y-recomendaciones)

## Introducción

Durante el ciclo de tareas llevadas a cabo con Aether SD-Core en el Proyecto 5G, se identificó que el NRF (Network Repository Function) presentaba **limitaciones** que dificultaban su correcta integración con funciones de red externas al núcleo 5G de Aether. Estas limitaciones se relacionaban con falta de campos necesarios en las solicitudes de registro de los elementos externos. Como resultado, surgió la necesidad de actualizar el código del NRF, reconstruirlo y someterlo a un proceso sistemático de pruebas que asegurara su correcto desempeño.

El **objetivo** de este informe es documentar el proceso de actualización del NRF en Aether SD-Core, detallando las acciones realizadas para configurar el entorno de desarrollo, modificar y compilar el código, y validar su funcionamiento en entornos controlados y progresivamente más complejos.

El **alcance** de este informe comprende:

- La configuración del entorno de desarrollo utilizado para trabajar con el código del NRF.

- Los cambios aplicados en el código fuente y su impacto en el comportamiento de la función.

- El proceso de build de cada función de red involucrada.

- Las pruebas realizadas inicialmente en un entorno con Docker.

- La adaptación de la configuración para su despliegue en Kubernetes con Aether OnRamp.

- La recolección de evidencia de los cambios realizados, mediante el análisis de logs obtenidos en pruebas de simulación.

Con ello, el documento proporciona una visión clara y ordenada del proceso seguido para resolver el problema identificado en el NRF, garantizando la trazabilidad de los cambios y su validación en diferentes escenarios.

---

## Configuración del entorno de desarrollo

Se hicieron forks de algunos de los repositorios de Aether, los cuales se pueden encontrar en [este enlace](https://github.com/orgs/networkgcorefullcode/repositories).

GitHub repository for the ONF: [https://github.com/opennetworkinglab](https://github.com/opennetworkinglab) — Documentación de OnRamp y playbooks para desplegar Aether.

Las imágenes de Docker se guardan en Docker Hub, se pueden encontrar buscando 'network5gcore' en la plataforma.



### Pasos iniciales

La configuración de un entorno de desarrollo se realiza con el objetivo de desarrollar los cambios en el código del NRF y construir las imágenes de cada uno de los componentes de Aether, utilizando configuraciones propias para crear un entorno válido para el desarrollo.

El entorno de desarrollo está compuesto por una VM (Máquina Virtual) en Proxmox con las siguientes características:

- SO (Sistema Operativo) Ubuntu Server 24.04 LTS
- 12 GB de RAM
- 6 CPUs
- 100 GB de almacenamiento


### Instalaciones necesarias

Las instalaciones necesarias en el servidor son las siguientes:

- [Go](https://go.dev/doc/install)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download)
- Python versión >= 3.8

Comandos para instalar Go en Linux Ubuntu 22.04:

```bash
cd ~
wget https://go.dev/dl/go1.24.6.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.6.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```

### Clonar repositorios

Ejecutar los siguientes comandos para crear la carpeta principal del proyecto:

```bash
cd ~
mkdir aether-forks
cd aether-forks
```

Crear aquí el archivo `python_get_repos.py`:

```bash
touch python_get_repos.py
```

Copiar el siguiente código (para clonar todos los repositorios rápidamente):

```python
import requests
import os

user = "networkgcorefullcode"  # Reemplaza con el nombre de usuario u organización
url = f"https://api.github.com/users/{user}/repos?per_page=100"
repos = requests.get(url).json()

for repo in repos:
  os.system(f"git clone {repo['clone_url']}")
```

```bash
python3 python_get_repos.py
```


Después de que termine la ejecución del script, se obtendrá la estructura de carpetas que se muestra en la Figura 1.

![Estructura de carpetas después de clonar los repositorios](imgs/{3A4EB7A6-8BC8-4E09-89EB-5599B0EB2BB5}.png)

Figura 1. Carpetas de cada repositorio del proyecto.

Actualmente, la clonación de los repositorios incluye el repositorio `utilFiles`, donde se definen varios de los archivos mencionados en esta sección del informe. Para poder utilizar estos archivos, se debe copiar su contenido en la raíz donde se encuentran todos los demás repositorios. De esta manera, podrán utilizarse sin inconvenientes.

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
    echo "Error al ejecutar make all"
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

Este script copiará todos los builds de Go en una carpeta llamada `bin`, como se muestra en la Figura 2.

```bash
./get_builds.sh
```

![Binario de cada uno de los componentes](imgs/{BBEA4130-7110-4E58-8C59-AC1895312AAC}.png)

Figura 2. Carpeta `bin` con las carpetas correspondientes a cada binario compilado con Go.


### Ejecutar componentes individualmente

Para ejecutar los componentes individualmente y realizar pruebas en cada uno de ellos, se debe proceder de la siguiente manera:

1. Se debe asegurar que el componente que se desea ejecutar tenga su binario en la carpeta `bin`.
2. Abrir una terminal y navegar a la carpeta `bin`, donde se encuentran los binarios de los componentes.

```bash
cd ~/aether-forks/bin
```

3. Ejecutar el binario del componente deseado. Por ejemplo, si se requiere ejecutar el componente `amf`, se utiliza el siguiente comando:

```bash
./amf --cfg ~/aether-forks/configs_files/amfcfg.yaml
```

Este procedimiento se aplica a cada uno de los componentes que soporten una configuración inicial a través de un archivo de configuración YAML.

De esta forma, es posible probar cada componente de manera individual y verificar su funcionamiento antes de integrarlos en un entorno más complejo, como Docker o Kubernetes. Esta práctica resulta especialmente útil para el desarrollo y la depuración de cada componente por separado.

## Ejemplos de desarrollo

### NRF (NfProfile Update)

El NRF de las versiones estables de Aether, presenta problemas a la hora de integrar nuevos perfiles de red, debido a que faltaban campos que estan presentes en release más modernos del 3GPP. A continuación se describe el proceso de desarrollo para la actualización del NRF y del modelo NfProfile.

Después de estudiar todas las alternativas para la actualización del NRF se decidió que con actualizar el modelo NfProfile se podrían integrar los nuevos perfiles de red de manera más eficiente, simplemente se deberia hacer un nuevo release de openapi con este modelo actualizado y especificar en cada NF de Aether que deberían utilizar esta nueva versión de openapi actualizada.

Las NF de Aether proporcionan en el registro de su perfil el campo NfServices y en releases más modernos se debe indicar el campo NfServiceList, este campo lo puede completar el NRF en el proceso de registro.

#### Proceso de desarrollo

1. Se actualizó el modelo NfProfile en el repositorio de Aether.

Para esto se generó el openapi correspondiente utilizando el openapi-generator-cli, el cual se encuentra en el repo de openapi. En el repo de  openApiFiles/files están todos los .yaml necesarios para generar el openapi de cada componente. En la actualidad el *Release* 16 tiene todos los .yml organizados hasta el *Release* 16. Se puede ver en <https://www.3gpp.org/ftp/specs/archive/OpenAPI/Rel-16>.

En una maquina ubuntu 22.04 o superior debes tener lo siguiente siguiente:

- Java 11 o superior
- Descargar el openapi-generator-cli.jar desde <https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/7.6.0/openapi-generator-cli-7.6.0.jar>
- Ejecutar el comando `java -jar openapi-generator-cli.jar help` para verificar que se ha instalado correctamente.
- Generar el openapi de go con los siguientes comandos:

```bash
java -jar openapi-generator-cli.jar generate   -i <path_a_tu_yaml_del_servicio>   -g go   -o ./go-nrf-client
```

En openApiFiles/openApiGeneratorOutputs/go-nrf-client estan todos los archivos generados del cliente de go correspondiente al NRF para el servicio Nnrf_NFManagement.

Para actualizar el NfProfile se utilizó el modelo generado por openapi generator, al se cometió un error que fue copiar y pegar los nuevos modelos generados, como una opción de desarrollo más rápida, pero esto introdujo una serie de bugs que provocaban varios cambios en la estructura original de Aether. El openapi generator genera structs y métodos utilizando una plantilla en común, a veces el código que genera no era lo que necesitaba Aether y se hacían modificaciones manuales según las necesidades. Entonces para evitar estos bugs se actualizó el modelo manualmente, comprobando que solo se agregaran nuevas *struct* sin romper las anteriores definidas por Aether.

Puedes visitar los siguientes commits donde se hicieron todos los cambios necesarios en el modelo de openapi
![alt text](imgs/{B4EE9F0D-3841-465E-947F-8172C116CD38}.png)

2. Se creó un nuevo release de openapi con el modelo actualizado.

Este paso es sencillo simplemente se creó un nuevo tag en la rama main y se hizo un push con ese tag.

3. Se especifica en cada NF de Aether que deben utilizar la nueva versión de openapi

En cada NF de aether se agregaron las siguientes líneas en el archivo de los modulos de go:

![alt text](imgs/image.png)

Luego se hicieron builds de estas NF para testearlas en el entornos de desarollo

Por último se actualizó el NRF para que llenara el campo NfServiceList en caso de que no llegara ese campo por parte de la NF. Los cambios los puede ver aquí: [Enlace a commit](https://github.com/networkgcorefullcode/nrf/commit/aaaf24b2cec666c1d2bd6cb2b2ad068a4193848f)

### WebUI Update

El WebUI tiene la posibilidad de mostrar y manejar información a través de una página web, haciendo uso de inteligencia artificial se creo una página web para agregar esta posibilidad, también se hicieron modificaciones en como se construye la imagen de docker para habilitar esta funcionalidad. Puede acceder a esta WebUI utilizando el la dirección y puerto que le asigne para este servicio.

### AMF, SMF, NSSF update to release 2.0.0

Estos componentes aún no tenían integrado nuevas funcionalidades como el *polling* HTTP al WebUI para obtener parámetros de configuración. Aether recientemente implementó nuevas funcionalidades para estos componentes, se han actualizado manualmente para agregarle estas nuevas características, estos cambios aún estan en desarrollo y en sus ramas, donde serán probados y luego lanzados en nuevo *release*.


## Entorno de Docker

Para crear un entorno de desarrollo utilizando Docker, se puede emplear un archivo `docker-compose.yaml` que defina los servicios necesarios para ejecutar los componentes de Aether. A continuación, se muestra un ejemplo básico de cómo podría estructurarse este archivo:

En la carpeta `configs_files/` deben ubicarse los archivos de configuración YAML para cada componente, como `amfcfg.yaml`, `ausfcfg.yaml`, etc. Estos archivos deben contener la configuración específica de cada componente.

En el repositorio `utilFiles` existe actualmente una serie de archivos docker-compose y scripts que permiten levantar un entorno de Docker, según las configuraciones asociadas en `configs_files`.

Hasta el momento, todo se encuentra en fase de prueba; la configuración puede no ser completamente estable, por lo que cualquier corrección será bienvenida. El objetivo es disponer de un entorno de pruebas funcional, que facilite el desarrollo y permita comprobar los resultados, todo ello utilizando Docker.

## Comandos útiles

- Para detener todos los contenedores:

```bash
docker compose down
```

- Para ver los registros de un contenedor específico:

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

- Para reconstruir los contenedores después de realizar cambios en el código:

```bash
docker compose up -d --build
```

Si se requiere instalar utilidades adicionales como `vim`, `strace`, `net-tools`, `curl`, `netcat-openbsd` y `bind-tools` en un contenedor basado en Alpine Linux, se puede ejecutar el siguiente comando dentro del contenedor:

```bash
apk update && apk add --no-cache -U vim strace net-tools curl netcat-openbsd bind-tools
```

Este comando actualizará los índices de paquetes e instalará las herramientas necesarias sin guardar archivos temporales, manteniendo la imagen ligera.

Script para instalar herramientas en los contenedores del core 5G:

```bash
#!/bin/bash

# Lista de contenedores del core 5G según docker-compose
core5g_containers=(amf ausf nrf nssf pcf smf udm udr)

for c in "${core5g_containers[@]}"; do
  echo "Instalando herramientas en $c..."
  docker exec -it "$c" sh -c "apk update && apk add --no-cache -U vim strace net-tools curl netcat-openbsd bind-tools"
done
```

## Kubernetes Environment

En las siguientes secciones se abordarán configuraciones relacionadas a un entorno de Kubernetes, ambiente donde Aether fue diseñado para desplegarse.

## Aether OnRamp

### Imágenes de Docker

Para desplegar los componentes de Aether actualizados, se requiere disponer de las imágenes de cada NF. Para ello, se realizaron los siguientes pasos:

1. En el repositorio de cada NF se editó el archivo `VERSION` y se cambió a un valor personalizado, en este caso fue: `v1.2.1-new-dev`.

2. En el archivo `Makefile` se completaron las siguientes variables:

```makefile
DOCKER_REGISTRY ?= 192.168.12.15:8083/
DOCKER_REPOSITORY ?= omecproject/
```

- `DOCKER_REGISTRY` se configuró con el valor del registry privado de Docker que se encuentra desplegado en un servidor Nexus en los servidores de la empresa.
- `DOCKER_REPOSITORY` se configuró con el nombre del repositorio por defecto de Aether SD-Core.

3. Hacer un `docker build` para construir las imágenes y luego un `docker push` para subirlas al registry. Para hacer el *push* es necesario primero iniciar sesión en el Nexus con `docker login 192.168.12.15:8083`.

Todos estos pasos pueden adaptarse según la conveniencia de cada usuario, como usar una versión diferente o subir las imágenes a otro registry de Docker.

### Configuración del *values* de Helm

El archivo de *values* de Helm se encuentra en la siguiente ruta, partiendo desde el directorio raíz del repositorio de Aether OnRamp: `/deps/5gc/roles/core/templates/sdcore-5g-values.yaml`.

En este archivo se hicieron varias configuraciones:

1. El despliegue se configuró para descargar las imágenes actualizadas del *registry* privado en Nexus de la siguiente manera:

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

```bash
omecproject/5gc-<nombre del componente en minúscula>:<valor definido en el archivo VERSION del repo del componente>
```

- Por ahora los despliegues se han hecho manteniendo el plano de usuario original de Aether, como se puede observar las imágenes actualizadas de esa sección están definidas pero comentadas.

2. Se añadieron configuraciones para varias NFs (AUSF, UDM, UDR, NSSF, PCF) ya que sin la definición de su configuración daban problemas al inicializarse los contenedores, y fueron modificadas otras (WebUI, AMF, NRF, SMF) para poder desactivar el cifrado TLS y tener toda la comunicación en HTTP. En este documento no se detallará cada configuración debido a que sería demasiado extenso. Para una inspección completa puede acceder al archivo [aquí](https://github.com/networkgcorefullcode/aether-onramp/blob/main/deps/5gc/roles/core/templates/sdcore-5g-values.yaml)

3. Debido a que se están usando componentes de Aether más actualizados (no solo por este proyecto sino también por desarrolladores oficiales de Aether) existen procesos nuevos. Uno de ellos es que ahora las NFs hacen un *polling* periódico al **WebUI** a través del puerto `5001`. Es por eso que cada NF debe tener esta configuración:

```yaml
 webuiUri: "http://webui:5001"
```

El manifiesto del ***service*** del **WebUI** no tiene este puerto configurado, por lo que se hizo necesario aplicar una solución que permitiera exponer el puerto y hacer permanente este cambio. Para ello se utilizó [Kustomize](https://kustomize.io), que permite aplicar modificaciones a manifiestos de Kubernetes. Kustomize tiene definida en un archivo la nueva configuración para el *service* del WebUI y la aplica como un parche al manifiesto final que se renderiza con Helm. Este proceso está incluido en el despliegue del 5GC con Ansible.

## Kubernetes para desarrollo

### Requisitos

- Ubuntu Server versión 22.04 o superior
- Docker
- Kubernetes
- Go versión 1.24.4 o superior

---

### Instalación del entorno de Kubernetes

---

Para desplegar Aether se debe tener un entorno de Kubernetes, en el cual utilizando los diferentes charts de Helm se despliegan los diferentes servicios.

#### 1. **Preparar el servidor**

Requisitos mínimos para el servidor:

- 2 CPU (4 si se va a desplegar Aether)
- 4–8 GB de RAM (se recomienda 8 GB para SD-Core)
- 20 GB o más de almacenamiento
- Sistema operativo Ubuntu Server 22.04 LTS
- Seguridad: grupo de seguridad con los puertos necesarios abiertos (SSH, 6443, 80, 443, 22, etc.)

---

#### 2. **Configurar el entorno base**

```bash
# Actualizar los paquetes del sistema
sudo apt update && sudo apt upgrade -y

# Desactivar el intercambio de memoria (swap), requisito para kubeadm
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

#### 3. **Instalar Docker (container runtime)**

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

sudo usermod -aG docker $USER
```

---

#### 4. **Instalar Kubernetes (kubeadm, kubelet, kubectl)**

```bash
sudo apt-get update -y
# apt-transport-https puede ser un paquete dummy; si es así, se puede omitir.
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Si el directorio `/etc/apt/keyrings` no existe, debe crearse antes de ejecutar el comando curl. Véase la nota siguiente.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Este comando sobrescribe cualquier configuración existente en /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```

---

#### 5. **Inicializar el clúster (modo single-node para pruebas)**

```bash
# (Opcional) Utilice la dirección IP pública o privada como advertise address
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}') \
  --cri-socket=/var/run/containerd/containerd.sock
```

Este comando instalará un clúster compatible con Calico/Flannel utilizando el pod CIDR especificado.

Si se presentan problemas relacionados con la ausencia de `containerd.sock`, se recomienda realizar lo siguiente:

Verificar que containerd esté instalado:

```bash
which containerd
```

El resultado debe ser similar a:

```bash
/usr/bin/containerd
```

Asegurarse de que containerd esté en ejecución:

```bash
sudo systemctl status containerd
```

Si el servicio no está activo, se debe iniciar:

```bash
sudo systemctl start containerd
```

Comprobar que el archivo de configuración de containerd tenga habilitado el CRI.

Primero, ejecutar:

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```

Luego, editar el archivo:

```bash
sudo nano /etc/containerd/config.toml
```

Buscar la siguiente sección:

```toml
[plugins."io.containerd.grpc.v1.cri"]
```

Verificar que no esté comentada y que se encuentre habilitada. Además, debe contener:

```toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

Reiniciar containerd después de realizar los cambios:

```bash
sudo systemctl restart containerd
```

Verificar que el socket exista:

```bash
ls -l /var/run/containerd/containerd.sock
```

Debe aparecer como un archivo de tipo socket.

Intentar nuevamente la inicialización:

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}') \
  --cri-socket=/var/run/containerd/containerd.sock
```

---

#### 6. **Configurar el entorno para el usuario actual**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

#### 7. **Instalar red de pods (ej. Calico)**

```bash
# Usando Calico como red de pods
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

---

#### 8. **Permitir que el nodo actúe como master y worker (modo prueba)**

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

Esto es necesario si solo tienes **una máquina** y quieres que los pods de usuario (como Aether) se ejecuten ahí.

---

#### 9. **Verifica que todo está funcionando**

```bash
kubectl get nodes
kubectl get pods -A
```

Se debe observar una salida similar a la siguiente:

![alt text](imgs/{70E4BAB7-F16D-481C-AF22-A3AF4EC88405}.png)

Figura ##. Pods de Kubernetes en estado **Running**

### Trabajando con Helm

#### Instalando Helm

Ejecute los siguientes comandos para instalar Helm:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

#### Obtener y operar charts

Aether proporciona una serie de charts de Helm, los cuales pueden configurarse según las necesidades del proyecto. Estos charts se encuentran en:

- [https://charts.aetherproject.org](https://charts.aetherproject.org)
- [https://charts.onosproject.org](https://charts.onosproject.org)
- [https://charts.opencord.org](https://charts.opencord.org)
- [https://charts.atomix.io](https://charts.atomix.io)
- [https://sdrancharts.onosproject.org](https://sdrancharts.onosproject.org)
- [https://charts.rancher.io/](https://charts.rancher.io/)

Los repositorios de GitHub relevantes son los siguientes:

- ROC: [https://github.com/onosproject/roc-helm-charts](https://github.com/onosproject/roc-helm-charts)
- SD-RAN: [https://github.com/onosproject/sdran-helm-charts](https://github.com/onosproject/sdran-helm-charts)
- SD-Core: [https://github.com/omec-project/sdcore-helm-charts](https://github.com/omec-project/sdcore-helm-charts)

Para este proyecto, se crearon repositorios propios adaptados a las necesidades específicas:

- SD-Core: [https://github.com/networkgcorefullcode/helm-charts](https://github.com/networkgcorefullcode/sdcore-helm-charts)

Se recomienda ejecutar los siguientes comandos opcionales para añadir repositorios de Helm:

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

# Para revertir estos cambios:
helm repo remove stable
helm repo remove cord
helm repo remove atomix
helm repo remove onosproject
helm repo remove sdran
helm repo remove aether
helm repo remove cetic
helm repo remove bitnami
```

Para configurar el almacenamiento local, se deben ejecutar los siguientes comandos:

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Desde el directorio `helm-charts/`, se recomienda ejecutar:

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
cd ..
```

A continuación, se deben instalar los componentes principales:

```bash
helm install -n kube-system atomix atomix-1.1.2/chart
helm install -n kube-system onos-operator onos-operator
kubectl create namespace roc5g
kubectl create namespace sdcore
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset.yml
kubectl get crd network-attachment-definitions.k8s.cni.cncf.io
kubectl label node <node_name> node-role.aetherproject.org=omec-upf

cd ~
git clone https://github.com/containernetworking/plugins.git
cd plugins
sudo chmod +x build_linux.sh
./build_linux.sh
```

Para desplegar el core 5G, se debe ejecutar el siguiente comando desde el repositorio `helm-charts`:

```bash
helm install -n sdcore core5g sd-core
```

Si los servicios relacionados con MongoDB no funcionan correctamente, se debe verificar que los charts se hayan descargado como dependencias. Se recomienda ejecutar el siguiente comando en cada uno de los charts dependientes:

```bash
cd sd-core/charts

charts=("5g-control-plane" "5g-ran-sim" "bess-upf" "omec-control-plane" "omec-sub-provision")

for chart in "${charts[@]}"; do
  helm dependency build "$chart"
  helm dependency update "$chart"
done

cd ..
cd ..
```

El WebUI es un componente principal para el funcionamiento del core 5G de Aether. En este despliegue, su inicialización puede retrasarse y provocar problemas en los demás componentes. Para solucionar esto, se recomienda eliminar los pods de los componentes del core, excepto el WebUI, para que el deployment los reinicie y, con el WebUI en funcionamiento, el sistema opere correctamente. Los siguientes comandos permiten realizar esta acción:

```bash
# Listar todos los pods excepto el webui, mongodb, kafka y core5g-zookeeper:
kubectl get pods -n sdcore | grep -v webui | grep -v mongodb | grep -v kafka | grep -v core5g-zookeeper

# Eliminar todos los pods excepto los mencionados:
kubectl get pods -n sdcore | grep -v webui | grep -v mongodb | grep -v kafka | grep -v core5g-zookeeper | awk '{print $1}' | xargs kubectl delete pod -n sdcore
```

#### Utils


Para habilitar una instancia de mongodb express que provea una interfaz web para visualizar fácilmente la base de datos mongodb, se puede usar el siguiente manifiesto:
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

Ejecute el comando `apply` correspondiente:

```bash
kubectl apply -f <file_name> -n <name_space>
```

Revise los logs y verifique que todo funcione correctamente. Si se presentan errores en el despliegue, se debe revisar la configuración proporcionada en el Helm chart correspondiente.

Puertos de los servicios

| Servicio           | Puerto | Descripción                               |
|--------------------|--------|-------------------------------------------|
| webui              | 30001  | Interfaz para el Webui                    |
| mongodb-express    | 30081  | Interfaz web para interactuar con mongodb |


### Comandos para eliminar kubernetes

```bash
kubectl delete -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset.yml

kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

kubectl delete -f https://raw.githubusercontent.com/k8snetworkplumbingwg/sriov-network-device-plugin/v3.3/deployments/k8s-v1.16/sriovdp-daemonset.yaml

kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

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



Configurando k8s

La instalación de Kubernetes (K8S) está fuera del alcance de esta guía. Una vez que K8S esté listo en la máquina virtual, se debe instalar `Multus` y `sriov-device-plugin`.

```bash
# Install Multus
$ kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/release-3.7/images/multus-daemonset.yml

# Verify
$ kubectl get ds -n kube-system kube-multus-ds
# NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE
# kube-multus-ds   1         1         1       1            1

# Create sriov device plugin config
# Replace PCI address if necessary
$ cat <<EOF | kubectl apply -n kube-system -f -
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
sudo apt update
sudo apt install jq -y
$ kubectl get node  -o json | jq '.items[].status.allocatable'
# {
#   "attachable-volumes-aws-ebs": "25",
#   "cpu": "7",
#   "ephemeral-storage": "46779129369",
#   "hugepages-1Gi": "2Gi",
#   "hugepages-2Mi": "0",
#   "intel.com/intel_sriov_vfio_access": "1",
#   "intel.com/intel_sriov_vfio_core": "1",
#   "memory": "13198484Ki",
#   "pods": "110"
# }
```

Por último, copia el binario `vfioveth` de CNI en la ruta `/opt/cni/bin` dentro de la máquina virtual.

```bash
sudo wget -O /opt/cni/bin/vfioveth https://raw.githubusercontent.com/opencord/omec-cni/master/vfioveth
sudo chmod +x /opt/cni/bin/vfioveth
```

Instalando bess upf con Helm

En el repo helm-charts hay una rama llamada deploy-ec2, aqui estan los helm-charts modificados, para que todo funcione correctamente en la instancia EC2.

A continuacion edita overriding-values.yaml según tus ip y macs (utiliza vim o nano, tu editor de texto de preferencia).

Verás algo como esto:

```yml
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
```

```bash
$ kubectl create namespace bess-upf
$ helm install -n bess-upf bess-upf [path/to/helm/chart] -f overriding-values.yaml
$ kubectl get po -n bess-upf
# NAME    READY   STATUS    RESTARTS   AGE
# upf-0   4/4     Running   0          41h
```


## Pruebas de simulación y trazas de logs de registro

Utilizando las configuraciones realizadas en Aether OnRamp se pudo comprobar el correcto funcionamiento de las nuevas características implementadas en Aether, a continuación se detalla como se llevo a cabo este proceso.

### Configuraciones previas

Primeramente se debe desplegar el Core 5G configurando el archivo `vars/main.yaml` y el archivo `host.ini` con las IPs de los servidores donde seran desplegados Aether y el simulador UERANSIM.

`vars/main.yml`

```yml
###...otras configuraciones

core:

###...otras configuraciones

  amf:
    ip: "<aether_server_IP>"

ueransim:
  gnb:
    ip: "<ueransim_server_IP>"

###...otras configuraciones

```

`host.ini`

```bash
[all]
node1 ansible_host=<aether_server_IP> ansible_user=<host_user>
ansible_ssh_private_key_file=<path_to_ssh_private_key>
node2 ansible_host=<ueransim_server_IP> ansible_user=<host_user> ansible_ssh_private_key_file=<path_to_ssh_private_key>

[master_nodes]
node1

[worker_nodes]
#node2
#node3

[gnbsim_nodes]
#node1
#node2
#node4

[oai_nodes]
#node2
#node4

[ueransim_nodes]
node2
#node4

[srsran_nodes]
#node2
#node4
```

#### Despliegue de Aether  y UERANSIM

Para desplegar aether primeramente es necesario instalar Kubernetes con:

```bash
make aether-k8s-install
```

Luego se despliega Aether con:

```bash
make aether-5gc-install
```

Para desplegar UERANSIM se corre el comando:

```bash
make aether-ueransim-install
```

#### Simulación

Para correr la simulación con UERANSIM se debe entrar por ssh a la VM del simulador, y en la carpeta `ueransim/build` correr los siguientes comandos.

Para simular la `gnb`:

```bash
./nr-gnb -c ../config/custom-gnb.yaml
```

Resultado esperado del comando anterior:

```bash
UERANSIM v3.2.7
[2025-08-13 15:55:46.882] [sctp] [info] Trying to establish SCTP connection... (192.168.12.11:38412)
[2025-08-13 15:55:46.921] [sctp] [info] SCTP connection established (192.168.12.11:38412)
[2025-08-13 15:55:46.921] [sctp] [debug] SCTP association setup ascId[3]
[2025-08-13 15:55:46.921] [ngap] [debug] Sending NG Setup Request
[2025-08-13 15:55:46.926] [ngap] [debug] NG Setup Response received
[2025-08-13 15:55:46.926] [ngap] [info] NG Setup procedure is successful
[2025-08-13 15:56:03.299] [rrc] [debug] UE[1] new signal detected
[2025-08-13 15:56:03.301] [rrc] [info] RRC Setup for UE[1]
[2025-08-13 15:56:03.306] [ngap] [debug] Initial NAS message received from UE[1]
[2025-08-13 15:56:03.961] [ngap] [debug] Initial Context Setup Request received
[2025-08-13 15:56:04.955] [ngap] [info] PDU session resource(s) setup for UE[1] count[1]
[2025-08-13 15:56:08.164] [rls] [debug] UE[1] signal lost
[2025-08-13 15:56:12.658] [rrc] [debug] UE[2] new signal detected
[2025-08-13 15:56:12.658] [rrc] [info] RRC Setup for UE[2]
[2025-08-13 15:56:12.658] [ngap] [debug] Initial NAS message received from UE[2]
[2025-08-13 15:56:12.957] [ngap] [debug] Initial Context Setup Request received
[2025-08-13 15:56:13.355] [ngap] [info] PDU session resource(s) setup for UE[2] count[1]
```

Para simular el UE:

```bash
./nr-ue -c ../config/custom-ue.yaml
```

Resultado esperado del comando anterior:

```bash
UERANSIM v3.2.7
[2025-08-13 15:56:12.658] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2025-08-13 15:56:12.658] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2025-08-13 15:56:12.658] [nas] [info] Selected plmn[208/93]
[2025-08-13 15:56:12.658] [rrc] [info] Selected cell plmn[208/93] tac[1] category[SUITABLE]
[2025-08-13 15:56:12.658] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2025-08-13 15:56:12.658] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2025-08-13 15:56:12.658] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2025-08-13 15:56:12.658] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-08-13 15:56:12.658] [nas] [debug] Sending Initial Registration
[2025-08-13 15:56:12.658] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2025-08-13 15:56:12.658] [rrc] [debug] Sending RRC Setup Request
[2025-08-13 15:56:12.658] [rrc] [info] RRC connection established
[2025-08-13 15:56:12.658] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2025-08-13 15:56:12.658] [nas] [info] UE switches to state [CM-CONNECTED]
[2025-08-13 15:56:12.698] [nas] [debug] Authentication Request received
[2025-08-13 15:56:12.698] [nas] [debug] Received SQN [000000000022]
[2025-08-13 15:56:12.698] [nas] [debug] SQN-MS [000000000000]
[2025-08-13 15:56:12.770] [nas] [debug] Security Mode Command received
[2025-08-13 15:56:12.770] [nas] [debug] Selected integrity[3] ciphering[3]
[2025-08-13 15:56:12.958] [nas] [debug] Registration accept received
[2025-08-13 15:56:12.958] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2025-08-13 15:56:12.958] [nas] [debug] Sending Registration Complete
[2025-08-13 15:56:12.958] [nas] [info] Initial Registration is successful
[2025-08-13 15:56:12.958] [nas] [debug] Sending PDU Session Establishment Request
[2025-08-13 15:56:12.958] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-08-13 15:56:13.355] [nas] [debug] PDU Session Establishment Accept received
[2025-08-13 15:56:13.355] [nas] [info] PDU Session establishment is successful PSI[1]
[2025-08-13 15:56:13.361] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 192.168.100.2] is up.
```

Luego de que esten correctamente inicializados el `gnb` y el `ue` se puede hacer ping a google con:

```bash
./nr-binder <uesimtun_interface_IP> ping google.com
```

`uesimtun_interface_IP` es la IP de la interfaz `uesimtun` que se puede obtener con el comando:

```bash
ip a
```

#### Cambio en la configuracion del *service* `nrf`

Para que el UDM pueda acceder el puerto del *service* del NRF, es necesario cambiar su tipo a `NodePort`. Esto se puede hacer con las siguientes instrucciones:

```bash
kubebctl edit svc nrf -n aether-5gc
```

- Comando para editar el service del `nrf` en el *namespace* `aether-5gc`

Salida esperada:

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: sd-core
    meta.helm.sh/release-namespace: aether-5gc
  creationTimestamp: "2025-08-12T15:13:15Z"
  labels:
    app: nrf
    app.kubernetes.io/managed-by: Helm
    release: sd-core
  name: nrf
  namespace: aether-5gc
  resourceVersion: "2363477"
  uid: b98e0080-3133-42fd-b362-4c9f1502e15c
spec:
  clusterIP: 10.43.77.24
  clusterIPs:
  - 10.43.77.24
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: sbi
    nodePort: 4567
    port: 29510
    protocol: TCP
    targetPort: 29510
  selector:
    app: nrf
    release: sd-core
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Se debe cambiar el campo `spec.type` a `NodePort`.

> [!NOTE] Nota Importante
> Este cambio se realiza "en caliente" luego de que esta desplegado todo el cluster. Si por algún motivo se reinstala Aether este cambio se sobreescribirá por su configuración original y será necesario realizarlo nuevamente.

Después de realizar el cambio de configuración en el *service* del NRF, al ejecutar el comando `kubectl get svc -n aether-5gc`, se podrá observar que el servicio del NRF tiene un puerto mapeado.

#### Instalación de Docker y Docker Compose

Docker:

```bash
sudo apt update
sudo apt install -y docker.io
```

Para usarlo como usuario normal:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Docker-Compose:

```bash
sudo wget -c https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-`uname -s`-`uname -m` -O /usr/local/bin/docker-compose; sudo chmod +x /usr/local/bin/docker-compose
```

#### Despliegue del UDM de Open5GS

Clonar el siguiente repositorio para obtener todos los archivos necesarios:

```bash
git clone https://github.com/networkgcorefullcode/udm-open5gs-deploy.git
```

Nota: Este repositorio también incluye configuraciones para el AUSF, el UDR y el Webui de Open5GS para probar la integración de estas NFs con Aether. Por el momento este informe solo estará enfocado en el registro del UDM.

Antes de desplegar el UDM de Open5GS se debe editar su archivo de configuración para indicar la dirección de la URI del NRF de Aether. Dentro de la carpeta del repositorio clonado ejecutar el siguiente comando

```bash
nano udm/udm.yaml
```

Se debe cambiar la dirección de `udm.sbi.client.nrf.uri` por la dirección IP del servidor y el puerto en el que se mapea el *service* del NRF de Aether. A continuación se muestra un ejemplo:

```yaml
logger:
    file:
      path: /open5gs/install/var/log/open5gs/udm.log

sbi:
    server:
      no_tls: true
    client:
      no_tls: true

global:
  max:
    ue: MAX_NUM_UE

udm:
    hnet:
      - id: 1
        scheme: 1
        key: /open5gs/install/etc/open5gs/hnet/curve25519-1.key
      - id: 2
        scheme: 2
        key: /open5gs/install/etc/open5gs/hnet/secp256r1-2.key
    sbi:
      server:
        - address: UDM_IP
          port: 7777
      client:
        nrf:
          - uri: http://192.168.12.11:4567 # 4567 es el puerto que del svc NRF
#        scp:
#          - uri: http://SCP_IP:7777
```

Luego se puede desplegar el UDM de Open5GS en Docker, utilizando el siguiente comando:

```bash
docker-compose -f udm-deploy.yaml up -d
```

#### Evidencia de registro en los logs

Luego de completar el despliegue del UDM se pueden ver los logs del contenedor con el comando:

```bash
docker logs udm
```

Salida esperada:

```logs-UDM
Deploying component: 'udm'
Open5GS daemon v2.7.5-24-g8e286b6

08/13 19:57:51.599: [app] INFO: Configuration: '/open5gs/install/etc/open5gs/udm.yaml' (../lib/app/ogs-init.c:144)
08/13 19:57:51.599: [app] INFO: File Logging: '/open5gs/install/var/log/open5gs/udm.log' (../lib/app/ogs-init.c:147)
08/13 19:57:51.600: [sbi] INFO: Setup NF EndPoint(addr) [192.168.12.11:17548] (../lib/sbi/context.c:459)
08/13 19:57:51.601: [sbi] INFO: NF Service [nudm-ueau] (../lib/sbi/context.c:1994)
08/13 19:57:51.601: [sbi] INFO: NF Service [nudm-uecm] (../lib/sbi/context.c:1994)
08/13 19:57:51.601: [sbi] INFO: NF Service [nudm-sdm] (../lib/sbi/context.c:1994)
08/13 19:57:51.601: [sbi] INFO: nghttp2_server() [http://172.22.0.13]:7777 (../lib/sbi/nghttp2-server.c:439)
08/13 19:57:51.601: [app] INFO: UDM initialize...done (../src/udm/app.c:31)
08/13 19:57:51.608: [sbi] INFO: [cb54f7b2-787f-41f0-abbf-f1bb9ccfa54d] NF registered [Heartbeat:60s] (../lib/sbi/nf-sm.c:295)
08/13 19:57:51.609: [sbi] ERROR: HTTP ERROR Status : 400 (../lib/sbi/message.c:1528)
08/13 19:57:51.609: [sbi] WARNING: No links (../lib/sbi/nf-sm.c:391)
08/13 19:57:51.610: [sbi] ERROR: No http.location (../lib/sbi/nnrf-handler.c:912)
08/13 19:57:51.611: [sbi] ERROR: No http.location (../lib/sbi/nnrf-handler.c:912)
```

En la siguiente línea se puede ver el resultado del registro del UDM en el NRF de Aether.

```bash
08/13 19:57:51.608: [sbi] INFO: [cb54f7b2-787f-41f0-abbf-f1bb9ccfa54d] NF registered [Heartbeat:60s] (../lib/sbi/nf-sm.c:295)
```

Los logs del NRF se pueden obtener con el comando:

```bash
kubectl logs -n aether-5gc <NRF_pod_name>
```

Salida esperada:

```logs-NRF
2025-08-13T19:57:51.601Z DEBUG management/api_nf_instance_id_document.go:102 Deserialize json data in the struct NfProfile {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.601Z INFO producer/nf_management.go:64 Handle NFRegisterRequest {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.601Z DEBUG producer/nf_management.go:461 [NRF] In NFRegisterProcedure {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.601Z WARN context/management_data.go:73 PLMN config not provided by NF, using supported PLMNs from webconsole {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.602Z DEBUG context/management_data.go:78 Fetched PLMN list from webconsole: [{Mcc:208 Mnc:93}] {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.602Z INFO context/management_data.go:103 HeartBeat Timer value: 60 sec {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.602Z DEBUG context/management_data.go:435 NfServices is nil, setting NfServices from NfServiceList {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.602Z DEBUG context/management_data.go:443 finish the function nnrfNFManagementOption {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.606Z INFO context/management_data.go:484 urilist update {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.607Z INFO producer/nf_management.go:527 Create NF Profile  UDM {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.608Z INFO producer/nf_management.go:542 Location header:  http://nrf:29510/nnrf-nfm/v1/nf-instances/cb54f7b2-787f-41f0-abbf-f1bb9ccfa54d {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.608Z DEBUG producer/nf_management.go:70 register success {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.608Z INFO logger/logger.go:91 | 201 |   192.168.12.11 | PUT     | /nnrf-nfm/v1/nf-instances/cb54f7b2-787f-41f0-abbf-f1bb9ccfa54d |  {"component": "NRF", "category": "GIN"}
2025-08-13T19:57:51.608Z INFO producer/nf_management.go:182 Handle CreateSubscriptionRequest {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.608Z INFO producer/nf_management.go:123 Handle GetNFInstancesRequest {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.609Z ERROR producer/nf_management.go:127 Error in string conversion:  0 {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.609Z INFO logger/logger.go:91 | 400 |   192.168.12.11 | GET     | /nnrf-nfm/v1/nf-instances |  {"component": "NRF", "category": "GIN"}
```

En esta línea se puede ver la llegada de la solititud de registro del UDM de Open5GS:

```logs-NRF
08/13 19:57:51.608: [sbi] INFO: [cb54f7b2-787f-41f0-abbf-f1bb9ccfa54d] NF registered [Heartbeat:60s] (../lib/sbi/nf-sm.c:295)
```

Se activa el procedimiento de registro:

```logs-NRF
2025-08-13T19:57:51.601Z DEBUG producer/nf_management.go:461 [NRF] In NFRegisterProcedure {"component": "NRF", "category": "MGMT"}
```

Se obtienen las configuraciones de PLMN del `webconsole` ya que no fueron proporcionadas por la nueva NF (UDM)

```logs-NRF
2025-08-13T19:57:51.601Z WARN context/management_data.go:73 PLMN config not provided by NF, using supported PLMNs from webconsole {"component": "NRF", "category": "MGMT"}
2025-08-13T19:57:51.602Z DEBUG context/management_data.go:78 Fetched PLMN list from webconsole: [{Mcc:208 Mnc:93}] {"component": "NRF", "category": "MGMT"}
```

Se crea un `NFProfile` para la nueva NF

```logs-NRF
2025-08-13T19:57:51.607Z INFO producer/nf_management.go:527 Create NF Profile  UDM {"component": "NRF", "category": "MGMT"}
```

Registro exitoso:

```logs-NRF
2025-08-13T19:57:51.608Z DEBUG producer/nf_management.go:70 register success {"component": "NRF", "category": "MGMT"}
```

Luego del registro, el NRF recibe una petición por parte del UDM para conocer otras NFs del Core:

```logs-NRF
2025-08-13T19:57:51.608Z INFO producer/nf_management.go:123 Handle GetNFInstancesRequest {"component": "NRF", "category": "MGMT"}
```

El NRF de Aether encuentra un error (`Error in string conversion: 0`) al procesar la petición de descubrimiento del UDM. Como resultado, responde al UDM con un error **400 Bad Request**.

```logs-NRF
2025-08-13T19:57:51.609Z ERROR producer/nf_management.go:127 Error in string conversion:  0 {"component": "NRF", "category": "MGMT"}
```

El UDM no sabe qué hacer con este error. Esperaba una lista de servicios y en su lugar recibió un "Bad Request". Como no puede descubrir otros servicios, su inicialización falla.

```logs-UDM
08/13 19:57:51.609: [sbi] ERROR: HTTP ERROR Status : 400 (../lib/sbi/message.c:1528)
08/13 19:57:51.609: [sbi] WARNING: No links (../lib/sbi/nf-sm.c:391)
08/13 19:57:51.610: [sbi] ERROR: No http.location (../lib/sbi/nnrf-handler.c:912)
08/13 19:57:51.611: [sbi] ERROR: No http.location (../lib/sbi/nnrf-handler.c:912)
```

El proceso anterior se describe también en la siguiente figura:
![Registro del UDM (Open5GS) en el NRF (Aether)](imgs/registo-del-UDM(Open5GS)-en-NRF(Aether).png)

Figura 6. Registro del UDM (Open5GS) en el NRF (Aether)

## Evaluación final y recomendaciones

Con las modificaciones realizadas en el código fuente de Aether SD-Core se logró registrar un UDM de terceros (en este caso, el UDM de Open5GS) en el NRF de Aether. Este procedimiento fue validado mediante evidencias en los registros tanto del UDM como del NRF. Sin embargo, tras el registro se presentaron fallos de interoperabilidad entre los componentes. Resolver estos problemas requiere continuar modificando el código del NRF y de otros servicios relacionados para alcanzar una funcionalidad completa del UDM de terceros. No obstante, este esfuerzo no resulta eficiente en el caso del UDM de Open5GS, ya que no será el utilizado en un entorno de producción, lo que podría generar nuevas incompatibilidades con el UDM de ETECSA, que es el componente objetivo con el cual se busca la interoperabilidad real.
