# Cambios al NRF

## Índice

- [Introducción](#introducción)
- [Uso del NRF en escenarios SNPN](#uso-del-nrf-en-escenarios-snpn)
- [Servicios soportados por el NRF](#servicios-soportados-por-el-nrf)
- [Nnrf_NFManagement](#nnrf_nfmanagement)

## Introducción

Para la actualización de este componente nos basaremos en los estándares establecidos por 3GPP.

En la siguiente web podemos encontrar la información de los endpoints y esquemas de los datos enviados y recibidos: <https://jdegre.github.io/parser.html>

También utilizaremos el documento 29.510 del 3GPP release 19.

Primeramente veamos las caracteristicas soportadas por el NRF para ver posibles implementaciones

## Uso del NRF en escenarios SNPN

El NRF (Network Repository Function) se utiliza en un SNPN (Standalone Non-Public Network) en los siguientes puntos clave:

- Registro y descubrimiento de funciones de red dentro de la red privada (SNPN).
- Autenticación y gestión de usuarios, ya sea usando servidores AAA-S o elementos AUSF/UDM.
- Facilita el onboarding (registro inicial) de dispositivos, tanto en escenarios donde todo ocurre en la misma red como cuando se usan credenciales externas.
- Soporta escenarios sin roaming (solo dentro de la misma red) y escenarios donde se interactúa con otras redes (diferentes PLMN) para autenticación y servicios.

## Servicios soportados por el NRF

- Nnrf_NFManagement
- Nnrf_NFDiscovery
- Nnrf_AccessToken
- Nnrf_Bootstrapping

## Nnrf_NFManagement

El servicio Nnrf_NFManagement permite que una instancia de NF, SCP o SEPP en el PLMN de servicio registre, actualice o elimine su perfil en el NRF.

### Resumen de funcionalidades adicionales de Nnrf_NFManagement y manejo de datos compartidos

- Si se soporta la función de "Shared-Data-Registration", los perfiles registrados, actualizados o eliminados pueden incluir identificadores de datos compartidos. Estos datos pueden estar configurados localmente en el NRF mediante OAM, o ser gestionados por la propia NF. En despliegues donde los datos compartidos se configuran localmente, no es obligatorio que el servicio Nnrf_NFManagement gestione el registro, actualización o eliminación de estos datos.
- Los datos compartidos pueden ser utilizados por funciones de red (NF) que pertenecen a un mismo conjunto (NF Set) para optimizar la señalización con el NRF. Alternativamente, se pueden usar otros métodos como OA&M para gestionar estos datos, estén o no asociados a un NF Set.
- Si se soporta la función de "Shared-Data-Retrieval", se permite a los consumidores recuperar datos compartidos, suscribirse o cancelar la suscripción a notificaciones de cambios en estos datos y recibir avisos cuando los datos sean modificados.
- El servicio Nnrf_NFManagement también permite que una instancia de NRF registre, actualice o elimine su perfil en otro NRF dentro del mismo PLMN. Alternativamente, esto también puede hacerse mediante OA&M.
- Permite que una NF o un SCP se suscriban para recibir notificaciones sobre el registro, eliminación o cambios de perfil de instancias de NF (y sus servicios) o de instancias SEPP. También permite a un SCP suscribirse a cambios de otras instancias SCP.
- El perfil de una NF incluye parámetros generales de la instancia y, si aplica, parámetros de los diferentes servicios que expone.
- Un NRF puede estar configurado con uno o varios PLMN ID (MCC y MNC). Debe soportar el registro, actualización y eliminación de perfiles de instancias de funciones de red provenientes de cualquiera de estos PLMN ID.
- El servicio Nnrf_NFManagement permite recuperar la lista de instancias de NF, SCP o SEPP registradas, o el perfil de una instancia específica.
- También permite verificar si las NFs, SCPs y SEPPs registradas están operativas.

### Operaciones que soporta este servicio
- NFRegister: 
- NFUpdate:
- NFDeregister:
- NFStatusSubscribe:
- NFStatusNotify:
- NFStatusUnsubscribe:
- NFListRetrieval:
- NFProfileRetrieval:
- SharedDataRetrieval:

### Actualizacion del NRF Aether

Esta es la issue principal estaremos trabajando en el repo <https://github.com/networkgcorefullcode/nrf> y en el repo <https://github.com/networkgcorefullcode/openapi>

Sacaremos una rama donde se estara trabajando en esta nueva implementacion: el nombre de la rama es `feature/nrf-update`. Utilizar commits semanticos y por cada operacion que se implemente, hacer un commit con el mensaje correspondiente, tratar de no hacer commits grandes, si no que cada commit sea una operacion o un cambio especifico. 

En el NRF trabajaremos en la rama `feature/openapi-nrf-update`. En el repo de openapi trabajaremos en la rama de igual nombre `feature/openapi-nrf-update`.

Dividiremos la implementación en varias tareas, cada una representada por una issue específica. Estas tareas incluirán:

- [ ] Actualizacion de la struct NfProfile para incluir los nuevos campos necesarios.
- [ ] Si estos campos son nuevas structs, crear estas structs y sus correspondientes métodos de serialización/deserialización.
- [ ] No hay mas issue, agregar las nuevas tareas aqui

#### Descripcion de como se esta trabajando en cada tarea

##### Actualizacion de la struct NfProfile para incluir los nuevos campos necesarios

Para esto se genero el openapi correspondiente utilizando el openapi-generator-cli, el cual se encuentra en el repo de openapi. En el repo de nostros openApiFiles/files estan todos los .yaml necesarios para generar el openapi de cada componente. En la actualidad el release 16 tiene todos los .yml organizados hasta el release 16. Lo puedes ver en <https://www.3gpp.org/ftp/specs/archive/OpenAPI/Rel-16>.

En una maquina ubuntu 22.04 hacer lo siguiente
- Java 11 o superior
- wget
- Descargar el openapi-generator-cli.jar desde <https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/7.6.0/openapi-generator-cli-7.6.0.jar>
- Ejecutar el comando `java -jar openapi-generator-cli.jar help` para verificar que se ha instalado correctamente.
- Generar el cliente y servidor de go con los siguientes comandos:

```bash
java -jar openapi-generator-cli.jar generate   -i <path_a_tu_yaml_del_servicio>   -g go   -o ./go-nrf-client
```

En openApiFiles/openApiGeneratorOutputs/go-nrf-client estan todos los archivos generados del cliente de go correspondiente al NRF para el servicio Nnrf_NFManagement.

Con este cliente generado, se pueden realizar pruebas y desarrollos sobre el NRF de manera más sencilla.

Ahora iremos paso a paso acutualizando el codigo del repo openapi y el repo nrf.



##### Si estos campos son nuevas structs, crear estas structs y sus correspondientes métodos de serialización/deserialización

##### No hay mas issue, agregar las nuevas tareas aqui