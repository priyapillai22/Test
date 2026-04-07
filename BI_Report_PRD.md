# Product Requirements Document (PRD)
## BI Report — HR Payroll Dashboard
### Domino's Pizza Netherlands — Nmbrs Payroll System

**Document version:** 1.0  
**Date:** April 2026  
**Audience:** Business stakeholders, HR managers, Finance team, IT team

---

## What Is This Document?

This document describes **what** we want to build: a Business Intelligence (BI) reporting tool that shows HR and payroll information visually — as charts, tables, and summaries — so that managers and finance staff can understand payroll costs quickly, without having to open spreadsheets or run manual reports.

Think of it like a **live dashboard on a screen** that answers questions like:
- "How much did we spend on wages this week across all stores?"
- "Which store had the highest payroll cost?"
- "Did all stores successfully send their payroll data?"

---

## 1. Background — Why Are We Building This?

Right now, Domino's Netherlands runs payroll by:
1. Connecting to each store's computer system
2. Pulling wage data into an Excel file
3. Manually checking the results
4. Uploading files to the payroll platform (called **Nmbrs**)

This works, but it has limitations:
- You have to open Excel to see anything
- It is hard to spot trends over time (e.g. is payroll going up month-on-month?)
- There is no easy way to compare stores side by side
- Errors (e.g. a store that failed to upload) can be missed

**The BI report solves this** by turning the raw payroll data into easy-to-read visuals that anyone can understand at a glance.

---

## 2. Goals

| Goal | What Success Looks Like |
|---|---|
| Save time | Managers get payroll insights in seconds, not hours |
| Reduce errors | Failed store imports are flagged automatically |
| Improve visibility | Finance can see cost trends without asking IT |
| Enable comparisons | Stores and departments can be compared side by side |
| Support audits | A history of every payroll period is stored and searchable |

---

## 3. Who Will Use This? (Users)

| User | Role | What They Need From the Report |
|---|---|---|
| **HR Manager** | Oversees all staff payroll | Summary of total headcount and wage cost per period |
| **Finance Controller** | Manages budgets and costs | Breakdown of costs by department and store; trend over time |
| **Payroll Administrator** | Runs the weekly/monthly payroll process | Confirmation that all stores uploaded correctly; employee-level detail |
| **Store Manager** | Manages a single Domino's store | Payroll costs for their store only |
| **IT / Operations** | Keeps the system running | Which stores failed to connect or upload data |

---

## 4. What Data Does the Report Use?

The report reads from files that are already produced by the existing payroll tool:

| Data File | Plain English Description |
|---|---|
| `HR-payroll.csv` | The main payroll export — one row per employee per store, showing wages |
| `LooncomponentenVar.xlsx` | Breakdown of variable salary parts (e.g. bonuses, overtime) |
| `KostenverdelingLooncomponenten.xlsx` | How costs are split across departments and cost centres |
| Store master list (`.dat` file) | The list of all Domino's stores, with their store number and name |
| Managers lookup table | Which manager is responsible for each employee/department |

> **No new data needs to be created.** The report reads from files the payroll tool already produces.

---

## 5. What Will the Report Show? (Features)

### 5.1 Main Summary Dashboard
*For: HR Manager, Finance*

This is the **home screen** of the report. It shows:

- 📦 **Total Payroll Cost** — the total wage bill for the selected period
- 👥 **Total Headcount** — how many employees were paid
- ✅ **Stores Imported OK** — how many stores successfully sent their data
- ❌ **Stores Failed** — how many stores had errors (needs attention)
- 📊 **Bar chart** — payroll cost per store, so you can compare stores instantly
- 📈 **Line chart** — payroll cost over time, so you can spot trends

---

### 5.2 Department / Cost Centre Breakdown
*For: Finance Controller*

This page answers: *"Where exactly is the money being spent?"*

- A table showing each department, its cost code, and how much was paid out
- A bar chart of the top 10 most expensive departments
- Filters so Finance can drill down by store, country, or time period
- Overhead departments (internal codes starting with `099` or `99`) are shown separately so they don't inflate store labour costs

---

### 5.3 Employee Detail Report
*For: Payroll Administrator*

This is a detailed table that shows every employee and what they were paid. It includes:

- Employee number and name reference
- Which store they work at
- Which department they are in
- Their manager's name
- The pay period dates
- Each salary component (base pay, overtime, bonuses, etc.)
- Total cost for that employee

This page also **highlights employees who work at more than one store** (called "multi-store employees"), because their payroll needs special handling.

---

### 5.4 Multi-Store Employee Report
*For: Payroll Administrator, HR Manager*

Some employees work across multiple Domino's stores. This page shows:

- Which employees appear at more than one store
- Their original employee reference and the remapped reference used in the system
- The total cost across all their stores combined
- A flag if any discrepancy is detected

---

### 5.5 Data Quality / Operations Report
*For: IT / Operations, Payroll Administrator*

This page is the **health check** of the payroll run. It shows:

- A table of every store with its connection status (Connected / Failed)
- Whether the store's data was imported successfully (✅ / ❌)
- When the last successful run was completed
- A percentage: "X out of Y stores imported successfully"
- An alert if the same store has failed multiple times in a row

---

## 6. Filters — How Users Can Slice the Data

Every page of the report will have these filter options so users can focus on what matters to them:

| Filter | What It Does |
|---|---|
| **Date range** | Show data for a specific week, month, or custom period |
| **Country** | Filter by Netherlands, Belgium, France, Germany, Luxembourg, or Denmark |
| **Store** | Show data for one specific store or all stores |
| **Department** | Narrow down to one department or cost centre |
| **Manager** | Show only employees managed by a specific person |

---

## 7. What the Report Must NOT Do (Out of Scope)

To keep the project focused, the following are **not** part of this report:

- ❌ The report does not replace the existing Excel payroll tool — it reads from its output
- ❌ The report does not send data to Nmbrs — that is still done by the existing tool
- ❌ The report does not edit or update any payroll records
- ❌ The report does not show individual employee personal details (e.g. address, bank account)

---

## 8. Access & Permissions

Not everyone should see everything. Here is how access will be controlled:

| Who | What They Can See |
|---|---|
| Payroll Administrator | All reports, all stores, all employees |
| HR Manager | All summary and headcount reports; all stores |
| Finance Controller | Cost and department reports; no individual employee detail |
| Store Manager | **Their store only** — cannot see other stores' data |
| IT / Operations | Data quality report only |

---

## 9. Data Refresh — How Often Is the Report Updated?

| Scenario | How Often |
|---|---|
| Normal payroll run | Report updates automatically after each payroll run |
| Historical data | All past periods remain available and searchable |
| Failed store run | The report shows a warning; data for that store is flagged as incomplete |

---

## 10. Non-Functional Requirements (How the Report Should Behave)

These are requirements about *quality* rather than features:

| Requirement | Target |
|---|---|
| **Speed** | Each report page should load within 5 seconds |
| **Reliability** | The report should be available 99% of business hours |
| **Accuracy** | Numbers must match exactly what the payroll export file contains |
| **Mobile-friendly** | Should be viewable on a tablet (not just a desktop) |
| **Audit trail** | Each data load should be logged with a timestamp and record count |

---

## 11. Recommended Tool

Given that the team already uses Microsoft Excel and the data lives in Excel/CSV files, the recommended platform is:

> **Microsoft Power BI**

Reasons:
- Connects directly to Excel and CSV files (no extra setup needed)
- Part of the Microsoft 365 suite that Domino's likely already has
- Easy to share dashboards via a web browser — no software installation needed for viewers
- Has built-in row-level security (so store managers only see their store)

Alternative: Tableau or a web-based tool like Metabase, if Power BI is not available.

---

## 12. Acceptance Criteria — How Do We Know It Is Done?

The BI report is considered complete and ready for use when:

- [ ] The main dashboard shows correct total payroll cost, headcount, and store import status for any selected period
- [ ] Payroll cost per store bar chart matches figures in the source CSV file
- [ ] A store manager logging in can only see their own store's data
- [ ] Failed store imports appear as red/alert items on the operations page
- [ ] Multi-store employees are correctly identified and shown on the dedicated page
- [ ] The report refreshes automatically after a new payroll run without manual intervention
- [ ] Finance can export any table to Excel for further analysis
- [ ] The report loads in under 5 seconds on a standard office laptop

---

## 13. Glossary — Plain English Definitions

| Term | What It Means |
|---|---|
| **BI Report** | Business Intelligence report — a visual dashboard that turns data into charts and summaries |
| **Payroll** | The process of calculating and paying employee wages |
| **Nmbrs** | The online payroll software that Domino's uses to process wages |
| **Cost centre / Department** | A specific part of the business that costs are tracked against (e.g. Store 30542, Kitchen) |
| **Debtor number** | A unique ID number assigned to each employee in the payroll system |
| **Multi-store employee** | An employee who works at more than one Domino's store in the same pay period |
| **Period** | The time span covered by a payroll run (typically one week or one month) |
| **Import** | The process of sending payroll data from the store database into the Nmbrs system |
| **Cost allocation** | How the total wage cost is divided across different departments |
| **Row-level security** | A setting that limits what data a user can see based on who they are |

---

*End of Document*
