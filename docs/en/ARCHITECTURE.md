# AuroraPay Billing System Architecture

> **Note**: This document reflects the architecture of a portfolio solution. For enterprise environments, the use of queues (RabbitMQ/Kafka) and segregated microservices is recommended.

This document describes the technical architecture of the billing automation system, following **Clean Architecture** and **Modularity** principles.

## 1. Overview
The system aims to read an invoice database (Excel), process business rules based on dates (due date), and send email notifications (HTML) in an idempotent manner.

## 2. Directory Structure
```
src/
├── core/           # Domain Entities (Pure)
├── services/       # Business Rules and Use Cases
├── infrastructure/ # External Adapters (Excel, Email, Log, OS)
└── utils/          # Shared Utilities
```

## 3. Main Components

### 3.1 Core (`src/core`)
Defines the fundamental data structures of the system, without external dependencies.
- **Entities**: `Invoice`, `InvoiceItem`.
- **Enums**: `BillingAction`.

### 3.2 Services (`src/services`)
Contains the orchestration logic.
- **BillingManager**: 
  - Reads invoices via `ExcelAdapter`.
  - Determines the rule to be applied (`D-5`, `D0`, `D+3`) based on the current date.
  - Checks idempotency (if already processed today) via `LogAdapter`.
  - Delegates email sending to `EmailAdapter`.

### 3.3 Infrastructure (`src/infrastructure`)
Implements communication with the external world.
- **ExcelAdapter**: Reads `Regua_Cobranca_V2.xlsx` file and normalizes data from multiple tabs (`Customers`, `Invoices`, `Items`) into domain objects.
- **EmailAdapter**: Manages HTML templates, dynamic variable replacement, and SMTP connection with Gmail.
- **LogAdapter**: Maintains an audit log (`execution_log.csv`) to ensure idempotency.

## 4. Execution Flow
1. **Start**: `main.py` is triggered (can receive `--test`, `--rule` flags).
2. **Setup**: Instantiation of adapters and dependency injection into `BillingManager`.
3. **Reading**: `BillingManager` requests invoices from `ExcelAdapter`.
4. **Processing** (Loop per Invoice):
   - Calculates Delta of days (Today - Due Date).
   - Defines Rule (e.g., `D0`).
   - If `--test` inactive: Checks if rule has already been executed today for that ID.
5. **Action**:
   - `EmailAdapter` loads corresponding template.
   - Generates dynamic items table.
   - Triggers email via SMTP.
6. **Log**: Result (SUCCESS/FAILED) is recorded in the CSV.

## 5. Design Patterns Used
- **Dependency Injection**: `BillingManager` receives adapters in the constructor.
- **Adapter Pattern**: Isolation of external libraries (`pandas`, `smtplib`).
- **Repository Pattern** (Simplified): Data reading abstracted in `ExcelAdapter`.
