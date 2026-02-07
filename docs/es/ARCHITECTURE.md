# Arquitectura del Sistema de Cobro AuroraPay

> **Nota**: Este documento refleja la arquitectura de una solución de portafolio. Para entornos enterprise, se recomienda el uso de colas (RabbitMQ/Kafka) y microservicios segregados.

Este documento describe la arquitectura técnica del sistema de automatización de cobros, siguiendo los principios de **Arquitectura Limpia (Clean Architecture)** y **Modularidad**.

## 1. Visión General
El sistema tiene como objetivo leer una base de datos de facturas (Excel), procesar reglas de negocio basadas en fechas (vencimiento) y enviar notificaciones por correo electrónico (HTML) de forma idempotente.

## 2. Estructura de Directorios
```
src/
├── core/           # Entidades de Dominio (Puras)
├── services/       # Reglas de Negocio y Casos de Uso
├── infrastructure/ # Adaptadores Externos (Excel, E-mail, Log, OS)
└── utils/          # Utilidades compartidas
```

## 3. Componentes Principales

### 3.1 Core (`src/core`)
Define las estructuras de datos fundamentales del sistema, sin dependencias externas.
- **Entidades**: `Invoice`, `InvoiceItem`.
- **Enums**: `BillingAction`.

### 3.2 Services (`src/services`)
Contiene la lógica de orquestación.
- **BillingManager**: 
  - Lee facturas a través de `ExcelAdapter`.
  - Determina la regla a aplicar (`D-5`, `D0`, `D+3`) en función de la fecha actual.
  - Verifica la idempotencia (si ya se procesó hoy) a través de `LogAdapter`.
  - Delega el envío de correo electrónico a `EmailAdapter`.

### 3.3 Infrastructure (`src/infrastructure`)
Implementa la comunicación con el mundo exterior.
- **ExcelAdapter**: Lee el archivo `Regua_Cobranca_V2.xlsx` y normaliza los datos de múltiples pestañas (`Clientes`, `Facturas`, `Artículos`) en objetos de dominio.
- **EmailAdapter**: Gestiona plantillas HTML, sustitución de variables dinámicas y conexión SMTP con Gmail.
- **LogAdapter**: Mantiene un registro de auditoría (`execution_log.csv`) para garantizar la idempotencia.

## 4. Flujo de Ejecución
1. **Inicio**: se activa `main.py` (puede recibir indicadores `--test`, `--rule`).
2. **Configuración**: instanciación de adaptadores e inyección de dependencias en el `BillingManager`.
3. **Lectura**: `BillingManager` solicita facturas al `ExcelAdapter`.
4. **Procesamiento** (Bucle por Factura):
   - Calcula el Delta de días (Hoy - Fecha de vencimiento).
   - Define la Regla (ej: `D0`).
   - Si `--test` está inactivo: verifica si la regla ya se ejecutó hoy para ese ID.
5. **Acción**:
   - `EmailAdapter` carga la plantilla correspondiente.
   - Genera una tabla de artículos dinámica.
   - Activa el correo electrónico a través de SMTP.
6. **Registro**: el resultado (SUCCESS/FAILED) se registra en el CSV.

## 5. Patrones de Diseño Utilizados
- **Dependency Injection**: `BillingManager` recibe adaptadores en el constructor.
- **Adapter Pattern**: aislamiento de librerías externas (`pandas`, `smtplib`).
- **Repository Pattern** (Simplificado): lectura de datos abstraída en el `ExcelAdapter`.
