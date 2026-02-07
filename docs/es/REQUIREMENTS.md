# Documento de Requisitos (REQ)

> **Alcance del Portafolio**: Los requisitos a continuación fueron dimensionados para demostrar la capacidad de análisis e implementación en Full Stack/Backend. En un escenario real, incluirían SLA, RGPD (LGPD) e Integración Bancaria vía API.

## 1. Objetivo
Automatizar el proceso de envío de notificaciones de cobro (facturas) por correo electrónico, garantizando puntualidad, personalización y reducción de la morosidad a través de una comunicación estructurada.

## 2. Requisitos Funcionales (RF)

| ID | Descripción | Regla de Negocio |
|----|-----------|------------------|
| **RF01** | **Lectura de Base de Datos** | El sistema debe leer un archivo Excel (`.xlsx`) que contenga múltiples pestañas (`Clientes`, `Facturas`, `Artículos`) y relacionar los datos. |
| **RF02** | **Cálculo de Plazos** | El sistema debe calcular automáticamente la diferencia de días entre la fecha actual y el vencimiento. |
| **RF03** | **Regla de Cobro** | Debe implementar las siguientes reglas de activación:<br>1. **D-5**: Recordatorio 5 días antes.<br>2. **D0**: Aviso el día del vencimiento.<br>3. **D+3**: Aviso de retraso 3 días después. |
| **RF04** | **Envío de Correo Electrónico** | El sistema debe enviar correos electrónicos en formato HTML con un diseño profesional y receptivo. |
| **RF05** | **Detalle de la Factura** | El cuerpo del correo electrónico debe contener una tabla con el detalle de los artículos/servicios cobrados y sus valores. |
| **RF06** | **Enlaces Dinámicos** | Debe generar botones funcionales para "Pagar Factura" y "Negociar" (este último solo para casos de retraso). |
| **RF07** | **Modo de Prueba** | Debe ofrecer un indicador `--test` para simular envíos sin validar la idempotencia diaria, marcando el asunto con `[TEST]`. |
| **RF08** | **Filtro de Ejecución** | Debe ofrecer un indicador `--rule` para permitir probar solo una regla específica (ej: solo D0). |

## 3. Requisitos No Funcionales (RNF)

| ID | Descripción | Criterio |
|----|-----------|----------|
| **RNF01** | **Idempotencia** | El sistema no puede enviar el mismo correo para la misma factura más de una vez en el mismo día (excepto en modo de prueba). |
| **RNF02** | **Portabilidad** | El sistema debe ejecutarse en cualquier entorno Windows/Linux con Python 3.8+ instalado. |
| **RNF03** | **Seguridad de Credenciales** | Las credenciales sensibles (contraseñas de correo electrónico) no deben estar en el código fuente, sino en variables de entorno (`.env`). |
| **RNF04** | **Auditoría** | Todas las acciones (éxito, falla u omisión) deben registrarse en un archivo de registro local (`execution_log.csv`). |
| **RNF05** | **Diseño UX/UI** | Los correos electrónicos deben seguir estándares de alto contraste y claridad visual (colores semánticos: Verde/Recordatorio, Azul/Hoy, Rojo/Atraso). |
| **RNF06** | **Mantenibilidad** | El código debe seguir una arquitectura modular (Clean Architecture) y los estándares PEP8. |
