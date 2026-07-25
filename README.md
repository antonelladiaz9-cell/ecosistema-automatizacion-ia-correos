# ecosistema-automatizacion-ia-correos


Entrega Final: Construcción de tu Ecosistema de Automatización IA
1. Caso de estudio
Sistema inteligente de lectura y resumen de correos de Gmail.
Todos los correos nuevos llegan a Gmail. La IA los analiza, genera un resumen, identifica si son importantes y guarda toda esa información en Notion. Luego el usuario decide si responder, archivar o ignorar el correo — el sistema no envía respuestas automáticas al cliente final.
Objetivo: automatizar la lectura de correos mediante IA para generar un resumen, clasificar la prioridad y sugerir una acción, reduciendo el tiempo de gestión del usuario.
2. Estructura del "Cerebro" (Base de datos)
Plataforma: Notion. Base de datos "Correos" con propiedades: Título, Remitente, Resumen IA, Prioridad, Acción, Estado. Estados implementados: Procesado por IA, Aprobado, Rechazado, Error IA.

### Base de datos en Notion
![Tabla de Notion con correos procesados](screenshots/Capturadepantalla2026-07-24212440.png)
![Tabla de Notion con correos procesados](screenshots/Capturadepantalla2026-07-24212453.png)
![Tabla de Notion con correos procesados](screenshots/Capturadepantalla2026-07-24212525.png)




3. Orquestación Lógica (el "Corazón")
Herramienta: n8n. Trigger inteligente con Gmail Trigger + nodo Filter (chequea que el correo no tenga la etiqueta Procesado-IA, para evitar bucles). Motor de IA con nodo AI Agent + Structured Output Parser (devuelve resumen, prioridad y acción en formato estructurado). Gestión de errores: el AI Agent tiene configurado "On Error → Continue Using Error Output"; la rama de error crea un registro en Notion con Estado = "Error IA".
### Canvas del workflow principal
![Workflow completo en n8n](screenshots/Capturadepantalla2026-07-24212608.png)
![Workflow completo en n8n](screenshots/Capturadepantalla2026-07-24212726.png)

4. Human-in-the-loop
Se agregó un nodo IF que evalúa si la Prioridad es "Alta". Si lo es, se dispara un nodo Slack ("Send and Wait for Response") que notifica al canal general-ia con Asunto, Remitente y Resumen IA, y espera la aprobación humana antes de continuar. Un segundo IF procesa la respuesta (approved: true/false) y actualiza el Estado en Notion a "Aprobado" o "Rechazado".
### Human-in-the-loop en Slack
![Mensaje de aprobación en Slack](screenshots/Capturadepantalla2026-07-24212823.png)
![Mensaje de aprobación en Slack](screenshots/Capturadepantalla2026-07-24212918.png)


5. Salida (canal adicional)
Se implementó un segundo workflow ("Resumen diario de correos") con Schedule Trigger (corre una vez al día) → Notion (Get Many, filtra Estado = Aprobado) → nodo Code (arma un HTML con la lista) → Gmail (envía el resumen por correo). Esto cumple el requisito de canal de salida adicional, coherente con el diseño del caso de uso (el sistema nunca contacta al cliente final automáticamente).

### Workflow de resumen diario
![Canvas del resumen diario](screenshots/Capturadepantalla2026-07-24213004.png)
![Canvas del resumen diario](screenshots/Capturadepantalla2026-07-24213025.png)
![Canvas del resumen diario](screenshots/Capturadepantalla2026-07-24213052.png)




6. Pruebas y seguridad
Antibucle: nodo Filter + Add Label al final de cada rama, evita reprocesar el mismo correo.
Manejo de errores: rama de error configurada con Notion como registro de fallos.
Prompt dinámico: el AI Agent usa variables del correo real (asunto, remitente, contenido), no texto fijo.
Filtro de tipos: la condición de prioridad compara texto (property_prioridad) contra texto ("Alta"), sin mezclar tipos.
### Historial de ejecuciones
![Executions con casos de éxito y error](screenshots/Capturadepantalla2026-07-24213141.png
)

