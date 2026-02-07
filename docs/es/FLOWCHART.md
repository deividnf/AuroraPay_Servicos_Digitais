# Diagrama de Flujo del Proceso

> **Visión General**: Flujo lógico detallado de la automatización AuroraPay.

```mermaid
flowchart TD
    %% Class Definitions
    classDef startEnd fill:#1A237E,stroke:#0D47A1,color:#ffffff,stroke-width:2px;
    classDef process fill:#E3F2FD,stroke:#2196F3,color:#0D47A1,stroke-width:1.5px;
    classDef io fill:#E8F5E9,stroke:#4CAF50,color:#1B5E20,stroke-width:1.5px;
    classDef decision fill:#FFF3E0,stroke:#FF9800,color:#E65100,stroke-width:1.5px;
    classDef error fill:#FFEBEE,stroke:#F44336,color:#B71C1C,stroke-width:1.5px;

    %% Nodes
    Start([Inicio del Proceso]):::startEnd
    End([Fin del Proceso]):::startEnd

    subgraph InputPhase ["1. Fase de Entrada"]
        direction TB
        ReadExcel["Lectura de Excel (V2)"]:::io
        AdapterExcel["ExcelAdapter (Normalización)"]:::process
        Sheets{Validación de Pestañas}:::decision
    end

    subgraph LogicPhase ["2. Fase de Inteligencia (Reglas)"]
        direction TB
        Merge["Consolidación de Datos<br/>(Cliente + Factura + Artículos)"]:::process
        LoopStart{{Para cada Factura ABIERTA}}:::process
        CalcDates["Cálculo de Delta (D-X)"]:::process
        CheckRule{¿Regla Activa?}:::decision
    end

    subgraph ActionPhase ["3. Fase de Ejecución (Email)"]
        direction TB
        TestCheck{¿Modo de Prueba?}:::decision
        CheckLog["Verificación de Idempotencia<br/>(Registro Diario)"]:::process
        SendEmail["EmailAdapter<br/>(HTML Dinámico)"]:::io
        StatusLog["Registro de Auditoría<br/>(SUCCESS / FAILED)"]:::io
    end

    %% Connections
    Start --> ReadExcel
    ReadExcel --> AdapterExcel
    AdapterExcel --> Sheets
    
    Sheets -->|OK| Merge
    Sheets -->|Falla| End
    
    Merge --> LoopStart
    LoopStart --> CalcDates
    CalcDates --> CheckRule

    CheckRule -->|D-5 / D0 / D+3| TestCheck
    CheckRule -->|Inactiva| Skip["Ignorar"]:::process
    Skip --> LoopStart

    TestCheck -->|NO| CheckLog
    TestCheck -->|SÍ| SendEmail

    CheckLog -->|Ya Enviado| Next["Próxima Factura"]:::process
    CheckLog -->|Pendiente| SendEmail

    SendEmail -->|SMTP Gmail| StatusLog
    StatusLog --> LoopStart
    Next --> LoopStart
    
    LoopStart -.->|Fin de la Lista| End

    %% Notes
    Note1["Control de Duplicidad<br/>basado en Factura_ID + Regla"]:::process
    CheckLog --- Note1
```

---
*Este diagrama de flujo representa la lógica de negocio implementada en la versión v2.1.*
