# Procedimiento Operativo Estándar (POE)

**Título**: Ejecución Diaria de la Regla de Cobro  
**Código**: POE-COB-001  
**Versión**: 1.0  
**Fecha**: 06/02/2026

> **Aviso de Uso**: Este procedimiento es ilustrativo para fines de portafolio. En producción real, este script se programaría mediante Cron/Task Scheduler o Airflow.

---

## 1. Objetivo
Estandarizar la ejecución del script de automatización de cobros, asegurando que los datos de entrada sean correctos y que el procesamiento ocurra de manera segura y auditable.

### 📋 Resumen Rápido (TL;DR)
| ¿Qué hacer? | ¿Dónde/Cómo? |
| :--- | :--- |
| **Preparar Datos** | Editar `data/input/Regua_Cobranca_V2.xlsx` |
| **Ejecutar** | Abrir terminal y ejecutar `python src/main.py` |
| **Validar** | Ver correos enviados y archivo `data/output/execution_log.csv` |

---

## 2. Prerrequisitos
1.  Python 3 instalado y configurado en el PATH.
2.  Archivo `.env` configurado con credenciales SMTP activas.
3.  Acceso a la carpeta `data/input/`.

## 3. Flujo de Procesamiento (Entrada/Salida)

### 3.1 Entrada de Datos
El sistema consume el archivo: `data/input/Regua_Cobranca_V2.xlsx`

**Estructura Obligatoria del Archivo:**
El archivo **DEBE** contener las siguientes pestañas y columnas. Los nombres de las columnas se normalizan (minúsculas/sin espacios), pero el orden ideal es:

1.  **Pestaña `Clientes`**:
    *   `cliente_id` (Clave Primaria, ej: CLI_01)
    *   `nome`
    *   `email`

2.  **Pestaña `Facturas`**:
    *   `fatura_id` (Clave Primaria, ej: FAT_001)
    *   `cliente_id` (Clave Foránea)
    *   `data_vencimento` (DD/MM/AAAA)
    *   `status`

3.  **Pestaña `Itens`**:
    *   `fatura_id` (Clave Foránea para vincular artículos a la factura)
    *   `descricao`
    *   `valor`

> **Nota**: El sistema acepta ajustes de columnas siempre que existan los nombres esenciales. Las columnas adicionales serán ignoradas.

### 3.2 Procesamiento
El script cruza la información:
1.  Lee todas las facturas abiertas.
2.  Busca los artículos correspondientes en la pestaña `Itens`.
3.  Busca los datos del cliente en la pestaña `Clientes`.
4.  Aplica la regla del día (D-5, D0, D+3).

### 3.3 Salida de Datos
*   **Correos electrónicos**: Enviados vía SMTP a los clientes.
*   **Registros (Logs)**: Registrados en `data/output/execution_log.csv` con Fecha, ID de Factura, Regla y Estado.

---

## 4. Instrucciones de Ejecución

### 4.1 Operación de Rutina (Producción)
Para ejecutar la regla oficial del día:
```bash
python src/main.py
```
*   El sistema verificará el log para evitar el envío de duplicados.

### 4.2 Simulación y Pruebas (QA)
Para verificar si una nueva base de datos es correcta sin enviar correos electrónicos duplicados reales ni afectar el log oficial:
```bash
python src/main.py --test
```

Para validar solo una regla específica (ej: comprobar si se están capturando los atrasados):
```bash
python src/main.py --test --rule D+3
```

## 5. Resolución de Problemas

| Síntoma | Causa Probable | Solución |
|---------|----------------|---------|
| "Cliente no encontrado" en el log | El ID del cliente en la pestaña `Facturas` no coincide con la pestaña `Clientes`. | Verifique la escritura de los ID en Excel. |
| Correo no enviado | Idempotencia activa (ya se ejecutó hoy). | Use `--test` para forzar o elimine la fila del log. |
| "Credenciales inválidas" | Archivo `.env` incorrecto. | Verifique la contraseña de la aplicación de Google en el `.env`. |
