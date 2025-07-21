# Logros durante NRFUPDATE

## Introducción

Este documento detalla los logros alcanzados durante la actualización del NRF (Network Repository Function)

1- Se actualizo el openapi correspondiente al servicio Nnrf_NFManagement, incluyendo los nuevos endpoints y esquemas de datos según las especificaciones del 3GPP release 19 2024 mes 12

2- Se hicieron cambios en el NRF para empezar a implementar este nuevo openapi, se logro correr en un entorno de docker el NRF modificado, el cual hasta ahora creo los endpoints sin dificultades, ahora toca actualizar los endpoints y como los maneja, despues hacer test manuales para verificar que todo funciona correctamente.
