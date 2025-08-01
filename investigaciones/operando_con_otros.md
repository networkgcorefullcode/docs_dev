# Operando con otros (5G/4G)

## Introducción

En la etapa actual del proyecto, se quiere implementar la capacidad de que un core, "basado" en Aether, pueda interactuar con otros componentes de otros core, para soportar funcionalidades avanzadas como roaming. Otra implemetación que se quiere lograr es poder utilizar un core 4G de Etecsa para obtener información de subscriptores que intenten autenticarse en la red 5G.

En los estudios realizados, determinamos que hay un soporte para estas funcionalidades, pero que hay posibilidades de que deben ser implementados por nosotros casos de uso en especifico, esto implica cambios en el código fuente de los componentes desarrollados en Aether, así como la posible creación de nuevos componentes que implementen las funcionalidades requeridas. Hemos estado estudiando cada uno de los componentes y experimentando haciendo pequeños cambios en el código fuente de varios componentes. Centrandonos en actualizar la sctruct del NfProfile con la que trabajan los componentes para dar soporte a que puedan interactuar con componentes de Cores más modernos, en especial el NRF.

Después de algunos análisis desarrollados durante todo este proceso, hemos podido concluir que una de las principales límitantes es que las NF desarrollados por Aether en su configuracion inicial tienen escaso soporte en cuanto a la selección y descubrimiento de las instancias. Esto trae consigo varias limitantes.

Ahora, nos proponemos a demostrar lo expuesto anteriormente en la parte 1 de este documento. Luego en las otras partes propondremos posibles soluciones a esta problematica y abordaremos otros temas relaciondos.

Antes de comenzar, le recomendamos leer, lo expuesto en el documento llamado registries.pdf, que encontrará en la misma carpeta.

## Parte 1

### Un breve ejemplo

Este ejemplo pretende ilustrar las limitaciones presentes en Aether.

Imaginemos que a nuestro core 5G intenta acceder un UE cuyos datos de subscripción se encuentran en el core 4G de Etecsa. ¿Cómo sabe nuestro core 5G que dicho subscriptor no tiene los datos en nuestros servidores?. Pues aquí entra en juego la selección y descubrimiento de las NF. Podríamos tener mapeadas las NF en relación a un rango de identifiación de los UE (Group ID). Y seleccionar instancias de acorde al identificador que nos envia el UE en los procesos de autenticación. Así podriamos tener todos los UDM que son capaces de interactuar con el HSS de Etecsa mapeados por un rango de identificadores y poder elegirlos en los procesos para que ese UE se pueda autenticar.

### Limitaciones

Después de analizar varios componentes podemos mencionar que el descubrimiento de las NF en Aether brinda la posibilidad de filtrar la busqueda que realizara el NRF a partir de ciertos parámetros, que le podemos indicar en la query que realizamos.

Estos parametros son especificados en la struct `SearchNFInstancesParamOpts` y son los siguientes:
ServiceNames
RequesterNfInstanceFqdn
TargetPlmnList
RequesterPlmnList
TargetNfInstanceId
TargetNfFqdn
HnrfUri
Snssais
Dnn
NsiList
SmfServingArea
Tai
AmfRegionId
AmfSetId
Guami
Supi
UeIpv4Address
IpDomain
UeIpv6Prefix
PgwInd
Pgw
Gpsi
ExternalGroupIdentity
DataSet
RoutingIndicator
GroupIdList
DnaiList
SupportedFeatures
UpfIwkEpsInd
ChfSupportedPlmn
PreferredLocality
AccessType
IfNoneMatch

Todos estos parámetros permiten realizar una busqueda más específica, devolviendo las NF que son de nuestro interes, según la operación a realizar.

Entonces, para nuestras NF, como UDM, UDR y demás, debemos poder editar su NfProfile, para indicar información como el Group ID, el rango de identificadores que pueden manejar, o el HSS con el que pueden interactuar. De esta forma, al realizar una query al NRF, podemos filtrar por el Group ID del UE que intenta autenticarse y obtener las NF que son capaces de interactuar con el HSS de Etecsa.

