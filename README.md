Ecosistema de Automatización IA — Gestión de Turnos Médicos

Entrega Final del curso de Automatización con IA. El sistema resuelve de punta a punta la solicitud de turnos médicos: un paciente escribe por Telegram en lenguaje natural, una IA extrae los datos del pedido, el sistema valida contra la base de pacientes, un humano aprueba o rechaza el turno por mail, y el paciente recibe la respuesta final — todo sin intervención manual salvo el punto de aprobación.

- Stack utilizado (4 categorías obligatorias)
Orquestador: n8n.
Base de datos: Airtable (tablas Pacientes, Turnos, Errores)
Procesamiento IA: Google Gemini (AI Agent + Structured Output Parser)
Canal de salida: Telegram (paciente) + Gmail (Human-in-the-Loop) 

Cómo funciona el flujo
1. **Trigger**: el paciente escribe un mensaje libre al bot de Telegram.
2. **IA**: un AI Agent (Gemini) extrae DNI, especialidad y fecha preferida, forzado a formato JSON por un Structured Output Parser.
3. **Validación**: si la IA no pudo identificar un DNI real, el flujo corta y le pide al paciente que reformule el mensaje (evita datos inventados, sin datos hardcodeados).
4. **Base de datos**: se busca al paciente en Airtable; si no existe, se le avisa y no se crea ningún turno.
5. **Creación del turno**: se crea el registro en estado `Pendiente aprobacion`.
6. **Human-in-the-Loop**: se envía un mail de aprobación por Gmail (botones Aprobar/Rechazar, con límite de espera de 24hs). El flujo queda pausado sin consumir recursos hasta la respuesta humana.
7. **Resolución**: según la decisión, se actualiza el `Estado` del turno en Airtable y se le informa el resultado al paciente por Telegram.
8. **Resiliencia**: cualquier falla en la llamada a Gemini, a Airtable o a Gmail se registra en la tabla `Errores` en vez de romper la ejecución completa, evitando bucles infinitos y comparando siempre tipos de datos correctos (DNI número vs número).

Enlaces:
- **Dashboard de Control — Turnos (KPIs por Estado): https://airtable.com/appRCJLS5w5n3D8Fq/pagv4mNY4xaT8KV9s
- Dashboard de Control — Errores (tasa de errores): https://airtable.com/appRCJLS5w5n3D8Fq/pag8Xfc7Y8wHoSTmV
- Video demo: https://drive.google.com/file/d/14xfx_8d_A63TArH8ifdw7C2sPNQf_hAC/view?usp=sharing

## Test de estrés realizado

El flujo fue probado con al menos 5 ejecuciones distintas: camino feliz (turno aprobado y turno rechazado por HITL) y camino infeliz (DNI inexistente, mensaje sin datos suficientes, y fallo forzado de una API), verificando que las rutas de error y los filtros de validación funcionan correctamente en cada caso. Las capturas de estas pruebas están incluidas en este repositorio.
