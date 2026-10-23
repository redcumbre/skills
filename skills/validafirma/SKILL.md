---
name: validafirma
description: "Úsala cuando te pidan integrar la firma electrónica, firmar documentos o conectar ValidaFirma a un sistema: qué es ValidaFirma, las nueve reglas que no se negocian y cómo pedir la guía con validafirma guia."
---

<!-- validafirma v23 sha256:7fbc2b8cde845846ecaefa84047146e8d48716bbb38eb796d27b7ebcd5e740ca -->
## ValidaFirma en tu proyecto

ValidaFirma es una plataforma de firma electrónica simple para Chile: tu sistema envía un PDF y los datos de quienes firman, cada firmante verifica su identidad y firma desde su celular, y tu sistema recibe el documento firmado. Quien te encargó esto no programa: háblale de su negocio.

Este skill es corto a propósito: las instrucciones para integrar se leen en vivo, cambian cuando cambia ValidaFirma y prevalecen sobre cualquier copia.

## Antes de todo

1. Comprueba que el CLI de ValidaFirma esté instalado: `validafirma --version`. Si no lo está, instálalo con `npm install -g @redcumbrecl/validafirma-cli`.
2. Corre `validafirma guia instrucciones` y síguelas paso a paso, desde el primero.
3. Si no puedes instalar el CLI o leer las instrucciones, no integres de memoria: dile a la persona en una frase qué falta.

## Las reglas que no se negocian

- RN-01: producción la decide la persona. Inicias `validafirma pasar-a-produccion` solo cuando ella te lo pide, y no ocurre sin su aprobación en el navegador; nunca lo simulas y nunca pides una llave de producción por otra vía. En producción, cada operación que crea, modifica o envía lleva dos confirmaciones: la tuya y la aprobación de la persona en el navegador. Por eso, en producción, opera solo con `validafirma api`; nunca llames la API a mano en producción ni corras el código del sistema con la llave de producción.
- RN-02: la llave y el secreto de producción nunca van al código, al repositorio, al navegador ni a la conversación: el sistema los lee de su configuración y el CLI los deja ahí con `pasar-a-produccion --instalar`, sin mostrarlos. No lees ni imprimes `keys/validafirma-produccion.json`, `VALIDAFIRMA_LLAVE_PRODUCCION` ni `VALIDAFIRMA_SECRETO_PRODUCCION`, y no pegas una llave ni un secreto en la conversación. La llave y el secreto de prueba sí puedes leerlos. Ninguna llave va al repositorio.
- RN-03: se avisa por webhook; no se consulta en bucle. Si consultas, nunca más de una vez cada 15 minutos y solo lo pendiente.
- RN-04: un documento terminado no cambia. Un 404 casi nunca es final.
- RN-05: cada webhook se verifica y se deduplica por su entrega.
- RN-06: lo de prueba no se mezcla con producción; mira `es_prueba` antes de actuar.
- RN-07: antes de cada operación corre `validafirma api describir <operación>`. El contrato prevalece sobre tu memoria.
- RN-13: el MCP no es una vía de integración; no sirve para integrar.
- RN-16: el PDF firmado se descarga desde el servidor del sistema con su llave, con `url_api_descarga` del aviso, o se envía al usuario `url_validacion` para que lo vea en el portal. `url_api_descarga`, la llave y la credencial nunca van al usuario ni a un correo, una página, una URL o un log. El código de verificación y `url_validacion` pueden ir al usuario, por correo o dentro de su sesión, pero no a una página abierta, un índice, un log ni una URL del sitio, y el `id` del documento tampoco. `url_descarga` del aviso está deprecada: deja de venir 30 días corridos después del aviso por correo, y la fecha exacta está en ese aviso y en la documentación del API.
