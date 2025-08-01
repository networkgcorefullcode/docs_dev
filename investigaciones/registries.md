# Documento sobre como se realiza el Discovery y Selection para distintas NF

## Introducción

Esto es una extracción de la especificación TS 23.501, que describe cómo se realiza el descubrimiento y la selección de las funciones de red (NF) en una red 5G. Para mayores detalles, recomendamos que visite la sección 6.3 de la especificación TS 23.501.

## SMF

Ver en la seccion 6.3.2 de TS 23.501.

Resumen:

La función de selección de SMF es compatible con AMF y SCP y se utiliza para asignar una SMF que gestionará la sesión de PDU. Los procedimientos de selección de SMF se describen en la cláusula 4.3.2.2.3 de TS 23.502 .

Si la AMF realiza el descubrimiento, utilizará la NRF para descubrir las instancias de SMF, a menos que la información de SMF esté disponible por otros medios, por ejemplo, configurada localmente en la AMF. La AMF proporciona información de ubicación del UE a la NRF cuando intenta descubrir las instancias de SMF. La NRF proporciona los perfiles de NF de las instancias de SMF a la AMF. Además, la NRF también proporciona el área de servicio de SMF de las instancias de SMF a la AMF. La función de selección de SMF de la AMF selecciona una instancia de SMF y una instancia de servicio de SMF según las instancias de SMF disponibles obtenidas de la NRF o según la información de SMF configurada en la AMF.

## UPF

Ver en la seccion 6.3.3 de TS 23.501.

## AUSF

Ver en la seccion 6.3.4 de TS 23.501.

En el caso del descubrimiento y selección basados en el consumidor NF, se aplica lo siguiente:

- La AMF y la NSWOF realizan la selección de AUSF para asignar una instancia de AUSF que realiza la autenticación entre el UE y la CN 5G en la HPLMN. La AMF y la NSWOF utilizarán la NRF para descubrir las instancias de AUSF, a menos que la información de AUSF esté disponible por otros medios, por ejemplo, configurada localmente en la AMF y en la NSWOF. La función de selección de AUSF en la AMF y en la NSWOF selecciona una instancia de AUSF según las instancias de AUSF disponibles (obtenidas de la NRF o configuradas localmente en la AMF).
- El UDM utilizará la NRF para descubrir las instancias de AUSF, a menos que la información de AUSF esté disponible por otros medios, por ejemplo, configurada localmente en el UDM. La UDM selecciona una instancia de AUSF basándose en las instancias de AUSF disponibles obtenidas del NRF o basándose en la información configurada localmente y la información almacenada (por la UDM) de una autenticación exitosa anterior.

La función de selección de AUSF en consumidores AUSF NF o en SCP debe considerar uno de los siguientes factores cuando estén disponibles:

1. Identificador de Red Local (p. ej., MNC y MCC, dominio) de SUCI/SUPI (por un consumidor NF en la red de servicio) junto con el NID seleccionado (proporcionado por la NG-RAN) en el caso de SNPN, Indicador de Enrutamiento y, opcionalmente, el identificador de Clave Pública de Red Local (p. ej., si el Indicador de Enrutamiento no es suficiente para proporcionar la granularidad del rango de SUPI).

NOTA 1: El UE proporciona el SUCI, que contiene el Indicador de Enrutamiento y el identificador de Clave Pública de Red Local, según se define en TS 23.003, a la AMF durante el registro inicial y a la NSWOF durante la autenticación de NSWO. La AMF puede proporcionar el Indicador de Enrutamiento del UE y, opcionalmente, el identificador de Clave Pública de Red Local a otras AMF, según se describe en TS 23.502.

NOTA 2: El uso del identificador de clave pública de red doméstica (IPC) para la detección de AUSF se limita a los casos en que los consumidores de NF de AUSF pertenecen a la misma PLMN que AUSF.

NOTA 3: En el caso de SNPN, si el UE proporciona un SUCI de tipo IMSI a la AMF, y el SUCI proporcionado por el UE o el SUPI derivado del SUCI corresponde a un SNPN atendido por la AMF, la AMF utiliza el NID seleccionado proporcionado por la NG-RAN junto con el ID de PLMN seleccionado (de IMSI) o el indicador de enrutamiento proporcionado por el UE dentro del SUCI para la selección de AUSF. En el caso de SNPN, si el UE proporciona un SUCI de tipo NSI a la AMF, la AMF utiliza el identificador de red doméstica (NID) y el indicador de enrutamiento de SUCI/SUPI para la selección de AUSF. Cuando el Indicador de Enrutamiento del UE se establece en su valor predeterminado, según lo definido en TS 23.003, el consumidor de AUSF NF puede seleccionar cualquier instancia de AUSF dentro de la red local para el UE.
2. ID Group de AUSF al que pertenece el SUPI del UE.

NOTA 4: La AMF puede inferir el ID de grupo de AUSF al que pertenece el SUPI del UE basándose en los resultados de los procedimientos de descubrimiento de AUSF con NRF. La AMF proporciona el ID de grupo de AUSF al que pertenece el SUPI a otras AMF, como se describe en TS 23.502.
3. SUPI; por ejemplo, la AMF selecciona una instancia de AUSF basándose en el rango de SUPI al que pertenece el SUPI del UE o basándose en los resultados de un procedimiento de descubrimiento con NRF utilizando el SUPI del UE como entrada para el descubrimiento de AUSF.

NOTA 5: En el caso de la incorporación mediante ON-SNPN, las instancias AUSF que admiten la incorporación de UE pueden registrarse en NRF o configurarse localmente en la AMF. La AMF en ON-SNPN puede detectar y seleccionar las instancias AUSF que admiten la incorporación de UE según el MCC y el MNC, o la parte del dominio, en el Identificador de Red Doméstica (SUCI/SUPI) proporcionado por el UE que se incorpora.

En el caso de la detección y selección delegadas en SCP, el consumidor de AUSF NF enviará todos los factores disponibles a SCP.

## AMF

Ver en la seccion 6.3.5 de TS 23.501.

## PCF

Ver en la seccion 6.3.7 de TS 23.501.

Detección y selección de PCF para una sesión de UE o PDU:

El descubrimiento y seleccion del PCF es implementado por el AMF, SMF, SCP y PCF para la PDU Session y siguen las siguientes clausulas:

- Puede haber múltiples PCF en una PLMN, cada uno con direcciones separadas.
- El PCF debe ser capaz de correlacionar la sesión de servicio AF establecida sobre N5 o Rx con la sesión PDU asociada (vinculación de sesión) gestionada sobre N7.
- Debe ser posible desplegar una red de modo que el PCF sirva únicamente a DN(s) específicos. Por ejemplo, el Control de Políticas puede habilitarse por DNN.
- La identificación única de una sesión PDU en el PCF debe ser posible en base al par (ID de UE, DNN), al par (Dirección(es) IP o MAC del UE, DNN) y al trío (ID de UE, Dirección(es) IP o MAC del UE, DNN).

Cuando el consumidor del servicio NF realiza el descubrimiento y la selección de PCF para un UE, se aplica lo siguiente:

- La AMF puede utilizar la NRF para descubrir las instancias PCF candidatas para un UE. Además, la información PCF también puede configurarse localmente en la AMF. La AMF selecciona una instancia PCF basándose en las instancias PCF disponibles obtenidas de la NRF o en la información configurada localmente en la AMF, según las políticas del operador.

En el caso sin roaming, la AMF selecciona una instancia PCF para la Asociación de Políticas de AM y la misma instancia PCF para la Asociación de Políticas de UE. En el caso de roaming, la AMF selecciona una instancia V-PCF para la Asociación de Políticas de AM y la misma instancia V-PCF para la Asociación de Políticas de UE.

La PCF para la sesión PDU selecciona una instancia (V-)PCF para la Asociación de Políticas de UE.

Los siguientes factores pueden considerarse en el descubrimiento y la selección de PCF para las políticas de Acceso y Movilidad y las políticas de UE:

- SUPI; La AMF selecciona una instancia de PCF según el rango de SUPI al que pertenece la SUPI del UE o según los resultados de un procedimiento de descubrimiento con NRF que utiliza la SUPI del UE como entrada para el descubrimiento de PCF.
- S-NSSAI. En caso de itinerancia, la AMF selecciona la instancia de V-PCF según la S-NSSAI de la VPLMN y la instancia de H-PCF según la S-NSSAI de la HPLMN.
- PCF ID Set
- PCF Group ID of the UE's SUPI.

NOTA 1: La AMF puede inferir el ID de grupo PCF al que pertenece el SUPI del UE, basándose en los resultados de los procedimientos de descubrimiento PCF con NRF. La AMF proporciona el ID de grupo PCF al que pertenece el SUPI a otros consumidores PCF NF, como se describe en TS 23.502.

- Capacidad de reemplazo de DNN de la PCF.
- Capacidad de reemplazo de slice de la PCF.
- Información de asistencia para la selección de PCF e ID(s) de PCF que atienden las sesiones PDU/conexiones PDN establecidas, recibidas del UDM. Si se recibe información de asistencia para la selección de PCF e ID(s) de PCF del UDM, la AMF selecciona la misma instancia de PCF que atiende la combinación de DNN y S-NSSAI, según lo indicado por la información de asistencia para la selección de PCF. Si se proporcionan múltiples combinaciones de DNN y S-NSSAI, la AMF selecciona la DNN y S-NSSAI mediante la configuración local. Si no se reciben los ID(s) de PCF (por ejemplo, si no se admite la interoperabilidad EPS), la AMF selecciona la instancia de PCF considerando los demás factores mencionados.
- Capacidad de entrega de URSP en EPS de la PCF.

Quedan por mencionar mas detalles, relacionados con el SMF y como realiza el descubrimiento y seleccion.

## UDM

El consumidor NF o el SCP realiza la detección de UDM para descubrir una instancia de UDM que gestione las suscripciones de los usuarios.

Si el consumidor NF realiza la detección y selección, este utilizará la NRF para descubrir las instancias de UDM, a menos que la información de UDM esté disponible por otros medios, por ejemplo, configurada localmente en los consumidores NF. La función de selección de UDM en los consumidores NF selecciona una instancia de UDM según las instancias disponibles (obtenidas de la NRF o configuradas localmente).

La función de selección de UDM es aplicable tanto al acceso 3GPP como al acceso no 3GPP. La funcionalidad de selección de UDM en el consumidor NF o en SCP debe considerar uno de los siguientes factores:

1. Identificador de Red Local (p. ej., MNC y MCC, dominio) de SUCI/SUPI, junto con el NID seleccionado (proporcionado por la NG-RAN) en el caso de SNPN, el Indicador de Enrutamiento del UE y, opcionalmente, el Identificador de Clave Pública de Red Local (p. ej., en caso de que el Indicador de Enrutamiento no sea suficiente para proporcionar la granularidad del rango de SUPI).
NOTA 1: El UE proporciona el SUCI a la AMF, que contiene el Indicador de Enrutamiento y el Identificador de Clave Pública de Red Local, según se define en TS 23.003 [19] durante el registro inicial. La AMF proporciona el Indicador de Enrutamiento del UE y, opcionalmente, el Identificador de Clave Pública de Red Local a otros consumidores NF (de UDM), según se describe en TS 23.502 [3].

NOTA 2: El uso del Identificador de Clave Pública de Red Local para el descubrimiento de UDM se limita a los casos en que los consumidores NF pertenecen a la misma PLMN que AUSF.

NOTA 3: En el caso de SNPN, si el UE proporciona un SUCI de tipo IMSI a la AMF, y el SUCI proporcionado por el UE o el SUPI derivado del SUCI corresponde a un SNPN atendido por la AMF, la AMF utiliza el NID seleccionado proporcionado por la NG-RAN junto con el ID de PLMN seleccionado (de IMSI) o el Indicador de Enrutamiento proporcionado por el UE dentro del SUCI para la selección de UDM. En el caso de SNPN, si el UE proporciona un SUCI de tipo NSI a la AMF, la AMF utiliza el Identificador de Red Doméstica y el Indicador de Enrutamiento de SUCI/SUPI para la selección de UDM.
Cuando el Indicador de Enrutamiento del UE se establece en su valor predeterminado, según se define en TS 23.003 [19], el consumidor UDM NF puede seleccionar cualquier instancia de UDM dentro de la red doméstica de SUCI/SUPI.
2. ID de Grupo UDM del SUPI del UE.
NOTA 4: La AMF puede inferir el ID de grupo UDM al que pertenece el SUPI del UE, basándose en los resultados de los procedimientos de descubrimiento UDM con NRF. La AMF proporciona el ID de grupo UDM al que pertenece el SUPI a otros consumidores UDM NF, como se describe en TS 23.502 [3].
3. SUPI o ID de grupo interno: el consumidor UDM NF selecciona una instancia UDM basándose en el rango de SUPI al que pertenece el SUPI del UE o en los resultados de un procedimiento de descubrimiento con NRF, utilizando el SUPI o el ID de grupo interno del UE como entrada para el descubrimiento UDM.
4. GPSI o ID de grupo externo: los consumidores UDM NF que gestionan la señalización de red no basada en SUPI/SUCI (por ejemplo, NEF) seleccionan una instancia UDM basándose en el rango de GPSI o ID de grupo externo al que pertenece el GPSI o el ID de grupo externo del UE, o en los resultados de un procedimiento de descubrimiento con NRF, utilizando el GPSI o el ID de grupo externo del UE como entrada para el descubrimiento UDM. En el caso de descubrimiento y selección delegados en SCP, el consumidor de NF deberá incluir uno de estos factores en la solicitud hacia SCP.

## UDR

Se pueden implementar múltiples instancias de UDR, cada una almacenando datos específicos o prestando servicio a un conjunto específico de consumidores NF, como se describe en la cláusula 4.2.5. En una implementación de UDR segmentada, diferentes instancias de UDR almacenan los datos para diferentes conjuntos y subconjuntos de datos, o para diferentes usuarios. Una instancia de UDR también puede almacenar datos de aplicación aplicables a cualquier UE, es decir, a todos los suscriptores de la PLMN.
Si el consumidor de servicios NF realiza el descubrimiento y la selección, utilizará la NRF para descubrir las instancias de UDR adecuadas, a menos que la información de la instancia de UDR esté disponible por otros medios, por ejemplo, configurada localmente en el consumidor NF.

La función de selección de UDR en los consumidores NF es aplicable tanto al acceso 3GPP como al acceso no 3GPP.

El consumidor NF o el SCP seleccionará una instancia de UDR que contenga información relevante para el consumidor NF, por ejemplo. UDM/SCP selecciona una instancia de UDR que contiene datos de suscripción, mientras que NEF/SCP (cuando se utiliza para acceder a datos de exposición) selecciona una UDR que contiene datos de exposición; o PCF/SCP selecciona una UDR que contiene datos de política o de aplicación.

Para la resolución del ID de grupo de NF correspondiente a un identificador de suscriptor, el consumidor de NF de UDR (p. ej., NRF, SCP) debe seleccionar una instancia de UDR compatible con el servicio Nudr_GroupIDMap.

Para los procedimientos de gestión de datos, la función de selección de UDR en los consumidores de NF de UDR considera el identificador del conjunto de datos de los datos que se gestionarán en UDR (véase la definición del servicio UDR en la cláusula 5.2.12 de TS 23.502 [3]). Además, la función de selección de UDR en los consumidores de NF de UDR debe considerar uno de los siguientes factores, cuando estén disponibles para el consumidor de NF de UDR, al seleccionar una UDR que almacene los conjuntos y subconjuntos de datos requeridos:

1. ID de grupo de UDR al que pertenece el SUPI del UE.
2. SUPI; p. ej., el consumidor NF de UDR selecciona una instancia de UDR según el rango de SUPI al que pertenece el SUPI del UE o según los resultados de un procedimiento de descubrimiento con NRF, utilizando el SUPI del UE como entrada para el descubrimiento de UDR.
3. GPSI o ID de grupo externo; p. ej., los consumidores NF de UDR seleccionan una instancia de UDR según el rango de GPSI o ID de grupo externo al que pertenece el GPSI o ID de grupo externo del UE o según los resultados de un procedimiento de descubrimiento con NRF, utilizando el GPSI o ID de grupo externo del UE como entrada para el descubrimiento de UDR.
4. Capacidad de UDR para almacenar datos de aplicación aplicables a cualquier UE (es decir, todos los suscriptores de la PLMN).

En caso de descubrimiento y selección delegados, el consumidor NF incluirá los factores disponibles en la solicitud a SCP.

## NRF

Los siguientes mecanismos pueden utilizarse para el descubrimiento de instancias de servicio NRF y sus direcciones de punto final:

- Los consumidores de NF o SCP pueden tener todas las instancias de servicio NRF y sus direcciones de punto final configuradas localmente.
- Los consumidores de NF o SCP pueden tener la dirección de punto final de un servicio de descubrimiento de NRF configurada localmente y utilizarla para descubrir las NRF y obtener sus perfiles de NF.
- Los consumidores de NF (p. ej., v-NRF) o SCP pueden tener las direcciones de punto final del servicio de arranque de NRF y utilizarlas para descubrir las instancias de servicio NRF y sus direcciones de punto final. El servicio de arranque de NRF es una API independiente de la versión, lo que puede ser especialmente útil en interfaces móviles.
- El consumidor de NF, p. ej., AMF, puede utilizar el servicio Nnssf_NSSelection para obtener la dirección de punto final de un servicio de descubrimiento de NRF para un segmento determinado.

## NSSF

Los siguientes mecanismos pueden utilizarse para el descubrimiento de instancias de servicio NSSF y sus direcciones de punto final en la red de servicio:

- El consumidor NF (es decir, AMF) o SCP puede tener todas las instancias de servicio NSSF y sus direcciones de punto final configuradas localmente.
- El consumidor NF (es decir, AMF) o SCP puede utilizar el servicio de descubrimiento NRF para descubrir los NSSF y obtener sus perfiles NF. En este caso, el descubrimiento y la selección de NSSF utilizan un NRF a nivel de PLMN (es decir, no específico de un segmento).

NOTA: El consumidor NF puede utilizar la información de ubicación, tal como se define en la cláusula 6.3.1.2, para seleccionar un NSSF. Para que un NSSF en una red visitada descubra y seleccione un NSSF en la red local, las instancias de servicios NSSF y sus direcciones de punto final del NSSF local se configuran localmente en el NSSF visitado o se descubren en función del FQDN autoconstruido como se especifica en TS 23.003.
