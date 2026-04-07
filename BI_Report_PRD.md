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

---

### 5.1 Main Summary Dashboard
*For: HR Manager, Finance*

This is the **home screen** of the report — the first thing you see when you open it. It gives you the big picture at a glance.

#### KPI Cards (the four headline numbers at the top)

| Card | What It Shows | How It Is Calculated |
|---|---|---|
| 💶 **Total Payroll Cost** | The total amount paid out to all employees in the selected period | Sum of the **Waarde (Amount)** column across all stores and departments |
| 👥 **Total Headcount** | How many unique employees were paid in the selected period | Count of distinct **Employee Numbers** in the data |
| ✅ **Stores Imported OK** | How many stores sent their payroll data successfully | Count of stores with an Import Result of `"V"` (success) |
| ❌ **Stores Failed** | How many stores had an error and did NOT send data | Count of stores with an Import Result of `"X"` (failed) |

> 💡 **Plain English:** Think of these four cards like the dashboard of a car — they show you the most important numbers immediately, without needing to read a report.

#### Chart 1 — Payroll Cost per Store (Bar Chart)

| Element | Description |
|---|---|
| **X-axis (horizontal)** | Store name (e.g. "Amstelveen Rembrandtweg") |
| **Y-axis (vertical)** | Total payroll cost in euros (€) |
| **Each bar** | Represents one store; taller bar = higher wage cost |
| **Colour coding** | Green = within expected budget range; Orange = above average; Red = significantly above average |
| **Purpose** | Lets you instantly compare which stores cost more or less to run |

#### Chart 2 — Payroll Cost Over Time (Line Chart)

| Element | Description |
|---|---|
| **X-axis (horizontal)** | Pay period (week or month, e.g. "Week 14 2026") |
| **Y-axis (vertical)** | Total payroll cost in euros (€) |
| **Each point on the line** | The total wage cost for all stores in that period |
| **Purpose** | Shows whether wage costs are going up, down, or staying stable over time |

---

### 5.2 Department / Cost Centre Breakdown
*For: Finance Controller*

This page answers: *"Where exactly is the money being spent — which department or part of the business?"*

#### Main Table — Cost by Department

Each row in this table is one department. Here is what each column means:

| Column | What It Shows | Example Value |
|---|---|---|
| **Department Code** | The unique code that identifies the department in the system | `30542`, `099-ADM` |
| **Department Name** | The human-readable name of the department | `"Kitchen"`, `"Delivery"`, `"Management"` |
| **Store Name** | Which Domino's store this department belongs to | `"Amstelveen Rembrandtweg"` |
| **Country** | The country the store is in | `NL`, `BE`, `FR`, `DE`, `LU`, `DK` |
| **Pay Period** | The week or month the cost was recorded for | `2026-W14` |
| **Period Start Date** | The first day of the pay period | `06/04/2026` |
| **Period End Date** | The last day of the pay period | `12/04/2026` |
| **Total Cost (€)** | The total wage amount charged to this department in this period | `€ 4,250.00` |
| **Employee Count** | How many employees had costs charged to this department | `12` |
| **Cost Code** | The Nmbrs cost code used for this allocation (always `U2101`) | `U2101` |

> ⚠️ **Note on Overhead Departments:** Department codes that start with `099` (5-digit) or `99` (4-digit) are internal overhead departments (e.g. head-office administration). These are **shown in a separate section** of this page so they do not make store labour costs look bigger than they really are.

#### Chart — Top 10 Most Expensive Departments (Bar Chart)

| Element | Description |
|---|---|
| **X-axis** | Department name |
| **Y-axis** | Total cost (€) |
| **Purpose** | Instantly see which 10 departments are spending the most |

---

### 5.3 Employee Detail Report
*For: Payroll Administrator*

This is the most detailed report — one row per employee per department per period. It shows exactly what each person was paid and how it was recorded.

#### Column-by-Column Specification

| Column | Column Name | What It Means | Example Value |
|---|---|---|---|
| 1 | **Debtor Number** | The unique payroll ID assigned to the employee in the Nmbrs system. **Note:** For employees who work at multiple stores, this number may have been changed to a "canonical" (master) version — the original number is shown in the Multi-Store column | `10042` |
| 2 | **Employee Number** | The employee's internal ID number from the store's database | `E00312` |
| 3 | **Department Code** | The code for the department this employee works in | `30542-KIT` |
| 4 | **Department Name** | The name of the department | `Kitchen` |
| 5 | **Store Name** | The name of the Domino's store | `Amstelveen Rembrandtweg` |
| 6 | **Country** | The country the store is in | `NL` |
| 7 | **Period Start Date** | The first day of the pay period this record covers | `06/04/2026` |
| 8 | **Period End Date** | The last day of the pay period (same as start date in weekly runs) | `06/04/2026` |
| 9 | **Salary Code** | The Nmbrs code identifying the type of pay (e.g. base wage, overtime, bonus) | `U2101` |
| 10 | **Amount (€)** | The euro value of this salary component for this employee in this period | `€ 352.00` |
| 11 | **Manager Name** | The name of the manager responsible for this employee and department | `Jan de Vries` |
| 12 | **Multi-Store Flag** | Shows `"Multi was (original number)"` if this employee works at more than one store and their debtor number was changed. Blank if not a multi-store employee | `Multi was 10041` |

> 🔶 **Rows highlighted in orange** indicate that the employee's debtor number was adjusted because they work across more than one store. This is normal and expected — it does not mean an error.

#### How to Read a Row (Plain English Example)

> *"Employee E00312 worked in the Kitchen department at Amstelveen Rembrandtweg during the week of 6 April 2026. Their manager is Jan de Vries. They were paid €352.00 in base wages (salary code U2101). They are not a multi-store employee."*

---

### 5.4 Multi-Store Employee Report
*For: Payroll Administrator, HR Manager*

Some employees work at more than one Domino's store in the same pay period. This causes a problem in payroll: if the same person appears twice (once per store), the system needs to know they are the same person. This report shows all such employees.

#### Column-by-Column Specification

| Column | Column Name | What It Means | Example Value |
|---|---|---|---|
| 1 | **Employee Number** | The employee's ID number | `E00312` |
| 2 | **Original Debtor Number** | The debtor number the employee had *before* any adjustment — as originally read from the store's database | `10041` |
| 3 | **Canonical Debtor Number** | The "correct" master debtor number assigned to this employee in the MultiMW lookup table — this is what gets uploaded to Nmbrs | `10042` |
| 4 | **Store 1 Name** | The first store this employee worked at | `Amstelveen Rembrandtweg` |
| 5 | **Store 2 Name** | The second (or additional) store this employee worked at | `Amsterdam Centrum` |
| 6 | **Total Cost Across All Stores (€)** | The sum of all wage costs for this employee across every store they worked at | `€ 615.00` |
| 7 | **Lookup Match Status** | Whether this employee was found in the MultiMW lookup table. `"Matched"` = remapping applied correctly; `"Not Found"` = employee appears at multiple stores but is not in the lookup table (needs investigation) | `Matched` |

> ⚠️ **Rows marked "Not Found"** need immediate attention from the Payroll Administrator. It means an employee is working at multiple stores but the system does not have a canonical debtor number for them. This must be added to the MultiMW lookup table before uploading to Nmbrs.

---

### 5.5 Data Quality / Operations Report
*For: IT / Operations, Payroll Administrator*

This page is the **health check** of the payroll run. It tells you which stores are working correctly and which ones had problems.

#### Summary Bar at the Top

| Indicator | What It Shows |
|---|---|
| **Total Stores** | How many stores are in the store master list |
| **Stores Reachable** | How many stores responded to a network ping |
| **Stores Imported OK** | How many stores had payroll data successfully extracted |
| **Import Success Rate** | Percentage: e.g. `"47 out of 50 stores (94%)"` |
| **Run Completed At** | The date and time the last payroll run finished |

#### Main Table — Status per Store

Each row is one Domino's store. Here is what each column means:

| Column | Column Name | What It Means | Possible Values |
|---|---|---|---|
| 1 | **Store Number** | The 5-digit code that uniquely identifies the store | `30542` |
| 2 | **Store Name** | The human-readable name of the store | `Amstelveen Rembrandtweg` |
| 3 | **Country** | The country the store is in, determined automatically from the store number range | `NL`, `BE`, `FR`, `DE`, `LU`, `DK` |
| 4 | **Network Status** | Was the store reachable over the network? | `Connected` ✅ or `Request timed out` ❌ or `Destination host unreachable` ❌ |
| 5 | **Import Status** | Did the store's payroll data import successfully? | `✅ OK` (value `"V"`) or `❌ Failed` (value `"X"`) |
| 6 | **Active in Run** | Was this store included in the last payroll run? | `Yes` or `No (deactivated)` |
| 7 | **Last Successful Import** | The date and time of the most recent successful data import for this store | `07/04/2026 08:22` |
| 8 | **Error Detail** | If the store failed, a brief description of what went wrong | `"SQL Server unavailable"`, `"Request timed out"`, blank if OK |

> 🔴 **Rows highlighted in red** indicate stores that failed. These need immediate attention — the Payroll Administrator should investigate the network connection or database availability for those stores.

#### Store Number → Country Reference

The country of each store is automatically determined from its store number. This table explains the logic:

| Store Number Range | Country | Example |
|---|---|---|
| 30540 – 31139 | 🇳🇱 Netherlands (NL) | `30542` → NL |
| 31140 – 31639 | 🇧🇪 Belgium (BE) | `31200` → BE |
| 31640 – 31659 | 🇱🇺 Luxembourg (LU) | `31650` → LU |
| 31660 – 33659 | 🇫🇷 France (FR) | `32000` → FR |
| 33710 – 34659 | 🇩🇪 Germany (DE) | `34000` → DE |
| 27010 – 27150 | 🇩🇰 Denmark (DK) | `27050` → DK |

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
