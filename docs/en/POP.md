# Standard Operating Procedure (SOP)

**Title**: Daily Execution of the Billing Rule Set  
**Code**: POP-COB-001  
**Version**: 1.0  
**Date**: 02/06/2026

> **Usage Notice**: This procedure is illustrative for portfolio purposes. In real production, this script would be scheduled via Cron/Task Scheduler or Airflow.

---

## 1. Objective
Standardize the execution of the billing automation script, ensuring that input data is correct and that processing occurs in a secure and auditable manner.

### 📋 Quick Summary (TL;DR)
| What to do? | Where/How? |
| :--- | :--- |
| **Prepare Data** | Edit `data/input/Regua_Cobranca_V2.xlsx` |
| **Execute** | Open terminal and run `python src/main.py` |
| **Validate** | Check sent emails and `data/output/execution_log.csv` file |

---

## 2. Prerequisites
1.  Python 3 installed and configured in PATH.
2.  `.env` file configured with active SMTP credentials.
3.  Access to `data/input/` folder.

## 3. Processing Flow (Input/Output)

### 3.1 Data Input
The system consumes the file: `data/input/Regua_Cobranca_V2.xlsx`

**Mandatory File Structure:**
The file **MUST** contain the following tabs and columns. Column names are normalized (lowercase/no spaces), but the ideal order is:

1.  **`Customers` Tab**:
    *   `customer_id` (Primary Key, e.g., CLI_01)
    *   `name`
    *   `email`

2.  **`Invoices` Tab**:
    *   `invoice_id` (Primary Key, e.g., FAT_001)
    *   `customer_id` (Foreign Key)
    *   `due_date` (DD/MM/YYYY)
    *   `status`

3.  **`Items` Tab**:
    *   `invoice_id` (Foreign Key to link items to the invoice)
    *   `description`
    *   `value`

> **Note**: The system accepts column adjustments as long as essential names exist. Extra columns will be ignored.

### 3.2 Processing
The script cross-references information:
1.  Reads all open invoices.
2.  Searches for corresponding items in the `Items` tab.
3.  Searches for customer data in the `Customers` tab.
4.  Applies the rule of the day (D-5, D0, D+3).

### 3.3 Data Output
*   **Emails**: Sent via SMTP to customers.
*   **Logs**: Recorded in `data/output/execution_log.csv` with Date, Invoice ID, Rule, and Status.

---

## 4. Execution Instructions

### 4.1 Routine Operation (Production)
To run the official rule set of the day:
```bash
python src/main.py
```
*   The system will check the log to avoid sending duplicates.

### 4.2 Simulation and Testing (QA)
To verify if a new database is correct without sending real duplicate emails or affecting the official log:
```bash
python src/main.py --test
```

To validate only a specific rule (e.g., check if overdue ones are being captured):
```bash
python src/main.py --test --rule D+3
```

## 5. Troubleshooting

| Symptom | Probable Cause | Solution |
|---------|----------------|---------|
| "Customer not found" in log | Customer ID in the `Invoices` tab does not match the `Customers` tab. | Check ID typing in Excel. |
| Email not sent | Idempotency active (already ran today). | Use `--test` to force or delete the row from the log. |
| "Invalid credentials" | Incorrect `.env` file. | Check Google app password in `.env`. |
