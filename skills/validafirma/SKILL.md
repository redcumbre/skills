---
name: validafirma
description: "Úsala cuando te pidan integrar la firma electrónica, firmar documentos o conectar ValidaFirma a un sistema: qué es ValidaFirma, las nueve reglas que no se negocian y cómo pedir la guía con validafirma guia."
---

<!-- validafirma v20 sha256:7d7291f18e27aa7e77024e2f6ed9528418cf1b5bb338a832c8f0b9f0902b0ee4 -->
## ValidaFirma en tu proyecto

ValidaFirma es una plataforma de firma electrónica simple para Chile: tu sistema envía un PDF y los datos de quienes firman, cada firmante verifica su identidad y firma desde su celular, y tu sistema recibe el documento firmado. Quien te encargó esto no programa: háblale de su negocio.

Este skill es corto a propósito: las instrucciones para integrar se leen en vivo, cambian cuando cambia ValidaFirma y prevalecen sobre cualquier copia.

## Antes de todo

1. Comprueba que el CLI de ValidaFirma esté instalado: `validafirma --version`. Si no lo está, instálalo con `npm install -g @redcumbrecl/validafirma-cli`.
2. Corre `validafirma guia instrucciones` y síguelas paso a paso, desde el primero.
3. Si no puedes instalar el CLI o leer las instrucciones, no integres de memoria: dile a la persona en una frase qué falta.

## Las reglas que no se negocian

- RN-01: lo real lo decide la persona. Nunca corres `pasar-a-real`, nunca lo simulas y nunca pides una llave real por otra vía. En real, cada operación que crea, modifica o envía lleva dos confirmaciones: la tuya y la aprobación de la persona en el navegador. Por eso, en real, opera solo con `validafirma api`; nunca llames la API a mano en real ni corras el código del sistema con la llave real.
- RN-02: la llave real y el secreto real nunca van al código, al repositorio, al navegador ni a la conversación: el sistema los lee de su configuración y la persona los copia ahí desde el panel. No lees ni imprimes `keys/validafirma-real.json` y no pegas una llave ni un secreto en la conversación. La llave y el secreto de prueba sí puedes leerlos. Ninguna llave va al repositorio.
- RN-03: se avisa por webhook; no se consulta en bucle. Si consultas, nunca más de una vez cada 15 minutos y solo lo pendiente.
- RN-04: un documento terminado no cambia. Un 404 casi nunca es final.
- RN-05: cada webhook se verifica y se deduplica por su entrega.
- RN-06: lo de prueba no se mezcla con lo real; mira `es_prueba` antes de actuar.
- RN-07: antes de cada operación corre `validafirma api describir <operación>`. El contrato prevalece sobre tu memoria.
- RN-13: el MCP no es una vía de integración; no sirve para integrar.
- RN-16: el código de verificación, `url_descarga` y el `id` del documento se protegen como el PDF y nunca van en una URL, página, correo o log público.
