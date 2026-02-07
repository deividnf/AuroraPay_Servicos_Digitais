# Requirements Document (REQ)

> **Portfolio Scope**: The requirements below were sized to demonstrate analytical and implementation capacity in Full Stack/Backend. In a real scenario, they would include SLA, GDPR (LGPD), and Banking Integration via API.

## 1. Objective
Automate the process of sending billing notifications (invoices) via email, ensuring punctuality, personalization, and reduction of delinquency through structured communication.

## 2. Functional Requirements (RF)

| ID | Description | Business Rule |
|----|-----------|------------------|
| **RF01** | **Database Reading** | The system must read an Excel file (`.xlsx`) containing multiple tabs (`Customers`, `Invoices`, `Items`) and relate the data. |
| **RF02** | **Deadline Calculation** | The system must automatically calculate the difference in days between the current date and the due date. |
| **RF03** | **Billing Rule** | Must implement the following triggering rules:<br>1. **D-5**: Reminder 5 days before.<br>2. **D0**: Notice on the due date.<br>3. **D+3**: Overdue notice 3 days after. |
| **RF04** | **Email Sending** | The system must send emails in HTML format with a professional and responsive layout. |
| **RF05** | **Invoice Detailing** | The body of the email must contain a table detailing the charged items/services and their values. |
| **RF06** | **Dynamic Links** | Must generate functional buttons for "Pay Invoice" and "Negotiate" (the latter only for overdue cases). |
| **RF07** | **Test Mode** | Must offer a `--test` flag to simulate sending without validating daily idempotency, marking the subject with `[TEST]`. |
| **RF08** | **Execution Filter** | Must offer a `--rule` flag to allow testing only a specific rule (e.g., only D0). |

## 3. Non-Functional Requirements (RNF)

| ID | Description | Criterion |
|----|-----------|----------|
| **RNF01** | **Idempotency** | The system cannot send the same email for the same invoice more than once on the same day (except in test mode). |
| **RNF02** | **Portability** | The system must run in any Windows/Linux environment with Python 3.8+ installed. |
| **RNF03** | **Credential Security** | Sensitive credentials (email passwords) must not be in the source code, but in environment variables (`.env`). |
| **RNF04** | **Auditing** | All actions (success, failure, or skip) must be recorded in a local log file (`execution_log.csv`). |
| **RNF05** | **UX/UI Design** | Emails must follow high contrast and visual clarity standards (semantic colors: Green/Reminder, Blue/Today, Red/Overdue). |
| **RNF06** | **Maintainability** | The code must follow a modular architecture (Clean Architecture) and PEP8 standards. |
