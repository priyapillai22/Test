# Functional Design Document
## HR-Payroll Nmbrs DBS — Amstelveen Rembrandtweg
**File:** `V4-HR-Payroll Nmbrs.DBS - 2 - Amstelveen Rembrandtweg.xlsm`  
**Version:** 4.00 | **Author:** Ronald Bax | **Last updated:** 20-03-2023  
**Purpose:** Extract payroll data from Domino's store SQL databases and produce import files for the **Nmbrs** payroll SaaS platform.

---

## 1. System Overview

This Excel macro workbook acts as an automated data-extraction and transformation bridge between:

- **Source:** Local MS SQL Server databases inside individual Domino's Pizza stores (accessed over the internal corporate network `dpev.net`)
- **Target:** The Nmbrs online payroll system (import files in a specific format)

The user operates the tool from a single control sheet ("Stores") using a set of action buttons. The workbook connects to each store's database, runs a stored procedure, retrieves payroll records, combines all store data, transforms it, and saves the result as importable Excel files and a CSV.

---

## 2. Worksheets and Their Roles

| Sheet name | VBA codename | Purpose |
|---|---|---|
| **Stores** | `wksStores` | Main control panel — all buttons, configuration cells, and the store list |
| **Result** | `wksResult` | Raw payroll data returned from all stores' SQL databases, combined |
| **Result2** | `wksResult2` | Alternate result sheet (same logic, for a second store group variant) |
| **Result3** | `wksResult3` | Alternate result sheet (same logic, for a third store group variant) |
| **Telling** | `wksTelling` | Pivot table showing employees who appear at multiple stores (duplicate detection) |
| **Managers** | `wksManagers` | Lookup table: employee number + department → manager name |
| **MultiMW** | `wksMultiMW` | Lookup table: multi-store employees and their canonical debtor numbers |
| **SysInfo** | `wksSysInfo` | Hidden developer sheet: system info, VBA module export/import utilities |

---

## 3. The Stores Sheet — User Interface

### 3.1 Configuration Cells (User Input)

| Cell | Label / Name | What the user enters | How it is used |
|---|---|---|---|
| **G2** | Processing date | Date for which payroll must be extracted (auto-set to today on open) | Passed as `@ForDate` parameter to the SQL stored procedure |
| **G3** | Store file path | Full path to the `.dat` store list file on disk | Read by "Reload Stores" to build the store table |
| **G4** | Standards file path | Full path to the external Standards Excel reference file | Read by "Reload Standards" to update wksStandard formulas |
| **I5** | Separator | Character used as column separator in the CSV output (e.g. `;`) | Passed to `SaveSheetToTXTFile` when writing `HR-payroll.csv` |
| **I6** | JustOneDay checkbox | TRUE = extract only the single day in G2; FALSE = extract the full period | Passed as `@JustOneDay` (1 or 0) to the SQL stored procedure |

> **Important validation:** If `I6 = TRUE` and `G2 = today's date`, the workbook blocks the operation with an error message: *"Can't use JustOneDay for TODAY. Change the date first."* This prevents accidentally pulling incomplete same-day data.

### 3.2 Status / Output Cells (Read-only for user)

| Cell | What it shows |
|---|---|
| **F9** | Start time of the last operation (hh:mm:ss) |
| **F10** | End time of the last operation (hh:mm:ss) |
| **G9** | Total elapsed duration of the last operation |
| **H8** | Count of stores connected (after Ping) / stores successfully imported (after Run) |
| **H10** | Average processing time per store |
| **G5** | Timestamp when Standards were last reloaded |
| **H7** | Timestamp when the last "Run in all stores" was completed |
| **I3** | Count of stores that were successfully imported (used in final message box comparison) |

### 3.3 Store List (Rows 2 onward)

| Column | Content | Meaning |
|---|---|---|
| **A** | `"V"` or blank | `"V"` = active, include this store; blank = skip |
| **B** | Store number | 5-digit store code (e.g. `30542`) — used to build the DNS hostname |
| **C** | Store name | Human-readable name of the store |
| **D** | Ping result | Filled after pressing [Ping]: `"Connected"` or an error description |
| **E** | Import result | Filled after pressing [Run]: `"V"` = data imported OK, `"X"` = failed |

---

## 4. Buttons and Their Behaviour

The workflow is sequential. A green circle indicator moves to the *next* button after each step is completed, guiding the user.

---

### Button 1 — 📂 Load Store File

**Macro:** `QAGeneric.LoadStoreFile`  
**When to click:** First-time setup, or when the store list file has moved to a new location.

**What happens:**
1. Opens a standard Windows file-open dialog.
2. User browses to and selects a `.dat` text file (the store master file).
3. The chosen full file path is written into cell **G3** of the Stores sheet.

**Input from user:** The `.dat` file to select.  
**Output:** Cell G3 updated with the file path. No other changes.

---

### Button 2 — 📂 Load Standards File

**Macro:** `QAGeneric.LoadStandFile`  
**When to click:** First-time setup, or when the Standards reference file has moved.

**What happens:**
1. Opens a standard Windows file-open dialog.
2. User selects an external Excel reference file.
3. The chosen full file path is written into cell **G4** of the Stores sheet.

**Input from user:** The external Standards Excel file to select.  
**Output:** Cell G4 updated with the file path. No other changes.

---

### Button 3 — 🔄 Reload Stores (`btnPing` gets the green circle next)

**Macro:** `QAGeneric.ReLoadStores`  
**When to click:** After loading the store file (Button 1), or whenever the store list may have changed.

**What happens step by step:**
1. Records start time in **F9**.
2. Clears the Result sheet (from row 3 downward).
3. Clears the store list table (column A–E, from row 11 downward) on the Stores sheet.
4. Saves the workbook.
5. Reads the `.dat` file at the path in **G3** via ADODB.Stream (encoding: `iso-8859-1`).
6. Parses each line:
   - Lines starting with `*` → 6-char prefix = inactive store (column A left blank).
   - Lines with a 5-char store number → column A = `"V"`, column B = store number, column C = store name.
   - Lines shorter than 6 chars are ignored.
7. Draws borders around the store table.
8. Sets **H8** formula: `=COUNTIF(A:A,"V")` — counts active stores.
9. Records reload timestamp in **G3** (date+time).
10. Auto-fits all columns, then sets fixed widths for columns E–I.
11. Marks the **[Ping]** button with a green circle (= next step).
12. Records end time / duration in **F10 / G9 / H10**.

**Input:**  
- `.dat` file (path from G3). Format: one store per line, first 5 chars = store number, remainder = store name.

**Output on screen:**  
- The Stores sheet store table is populated (columns A–C).
- H8 shows count of active stores.
- Processing time shown in F9/F10/G9/H10.

**No external files written.**

---

### Button 4 — 📡 Ping All Stores (`btnPing`)

**Macro:** `QAGeneric.PingInAllStores`  
**When to click:** After reloading stores, to verify which stores are reachable over the network before attempting data extraction.

**What happens step by step:**
1. Records start time in **F9**.
2. Clears the Result sheet.
3. For every row where column A = `"V"`:
   a. Converts the store number (column B) to a DNS hostname using the pattern:  
      `dp` + *lowercase country code* + *store number* + `.dpev.net`  
      Country is determined by store number range:

      | Store number range | Country | Example hostname |
      |---|---|---|
      | 30540 – 31139 | NL (Netherlands) | `dpnl30542.dpev.net` |
      | 31140 – 31639 | BE (Belgium) | `dpbe31200.dpev.net` |
      | 31640 – 31659 | LU (Luxembourg) | `dplu31650.dpev.net` |
      | 31660 – 33659 | FR (France) | `dpfr32000.dpev.net` |
      | 33710 – 34659 | DE (Germany) | `dpde34000.dpev.net` |
      | 27010 – 27150 | DK (Denmark) | `dpdk27050.dpev.net` |
      | 88xxx series | NL / BE / FR / DE / LU / DK | Country from specific case |

   b. Sends a WMI ping (`Win32_PingStatus`) to the hostname.
   c. If not connected: waits 1 second and pings again.
   d. Writes the result to **column D** of that store row:
      - `"Connected"` — store is reachable
      - `"Request timed out"` — store not responding
      - `"Destination host unreachable"` — routing failure
      - Various other WMI status descriptions
   e. If still not `"Connected"`: **clears column A** for that row → store is deactivated and will be skipped in the next step.
4. Sets **H8**: `=COUNTIF(D:D,"Connected")` — count of reachable stores.
5. Auto-fits column D.
6. Marks the **[Run In Stores]** button with a green circle.
7. Records end time / duration.

**Input:** None (reads store table from Stores sheet).  
**Output on screen:**
- Column D filled with `"Connected"` or error messages.
- Unreachable stores: column A cleared (deactivated).
- H8 shows count of reachable stores.

**No external files written.**

---

### Button 5 — 📋 Reload Standards (`btnStand`)

**Macro:** `QAGeneric.ReloadStandards`  
**When to click:** When the Standards reference file has changed or moved.

**What happens step by step:**
1. Records start time in **F9**.
2. Clears the Result sheet.
3. Checks that the file at path **G4** exists — if not, shows error and exits.
4. Scans the `wksStandard` sheet formulas to find external file references and replaces the old file path with the new one from G4 (`ReplaceRef`).
5. Finds the row in `wksStandard` with the label `"Standard"` in column A.
6. Fills formula(s) downward from that row (auto-fills relative references).
7. Records timestamp in **G5**.
8. Marks the **[Run In Stores]** button with a green circle.
9. Records end time / duration.

**Input:** External Standards Excel file (path from G4).  
**Output:** `wksStandard` sheet formulas updated. No external files written.

---

### Button 6 — ▶️ Run In All Stores (`btnRunInStores`) ← **Main Button**

**Macro:** `QAGeneric.RunInAllStores` → calls `wksResult.GetAndProcessData`  
**When to click:** After pinging (and optionally reloading standards). This is the core operation that extracts, combines, transforms, and exports all payroll data.

**What happens — full pipeline:**

#### Phase 1 — Preparation
1. Records start time in **F9**.
2. Clears the Result sheet (removes old data).
3. Saves the workbook (ensures a clean recovery point).

#### Phase 2 — SQL Data Extraction (per store)
4. Reads the processing date from **G2**, formats as `yyyy-mm-dd`.
5. Reads the separator from **I5** and JustOneDay flag from **I6**.
6. Builds the SQL call:
   ```sql
   EXEC DPEU.dbo.spDOM_ExtractPayrollNmbrsNL
     @ForDate = '<date from G2>',
     @Header  = 0,
     @JustOneDay = <0 or 1 from I6>
   ```
7. For every store where column A = `"V"` and column B is not empty:
   - Resolves store number → DNS hostname (same logic as Ping).
   - Connects to that store's SQL Server:
     - Provider: `SQLOLEDB.1`
     - User: `maint`
     - Database: `DPEU`
     - Server: `<hostname>`
   - Executes the stored procedure.
   - Appends returned rows to the **Result sheet** (starting from the next empty row below existing data).
   - Writes `"V"` in column E of the store row = success; `"X"` = failed.

#### Phase 3 — Multi-store Employee Fix
8. Looks up each employee record in the **MultiMW sheet** using the key `debtor_number + "-" + department_code` (column E of MultiMW).
9. If found: replaces the debtor number (column A of Result) with the canonical debtor number from MultiMW column F.
10. Highlights the changed cell in **orange** and writes `"Multi was (original_value)"` in column M of Result.

#### Phase 4 — Add Manager Column
11. Fills column **L** of Result with a VLOOKUP-based formula that looks up the manager name from the **Managers sheet** (columns C:D), using `employee_number & LEFT(department_code,5)` as the key (falls back to employee number alone if no department-specific match found).

#### Phase 5 — Save Raw CSV
12. Auto-fits all columns in Result.
13. Saves the Result sheet as:  
    📄 **`<workbook folder>\HR-payroll.csv`** (separator = character in I5).

14. Forces recalculation and refreshes all pivot caches.

#### Phase 6 — Cleanup Result Sheet
15. Deletes rows where column C (department code) starts with `099` (5-char) or `99` (4-char). These are internal overhead cost centers that should not be in the Nmbrs import.

#### Phase 7 — Build Nmbrs Import Sheets
16. Deletes any pre-existing `LooncomponentenVar`, `KostenverdelingLooncomponenten`, and `Temp` worksheets.
17. Copies the Result sheet → temporary sheet named **`Temp`**.
18. From `Temp`, removes all rows where column L (manager) is NOT empty → keeps only non-manager / non-overhead employees.
19. Creates two copies of `Temp`:
    - First copy → **`LooncomponentenVar`**
    - Second copy → **`KostenverdelingLooncomponenten`**

#### Phase 8 — Format LooncomponentenVar (Salary Variable Components)
20. Activates the `LooncomponentenVar` sheet and:
    - Deletes columns I through M.
    - Inserts a new header row 1: `A1 = "Import"`, `E1 = "Looncomponenten"`.
    - Clears column H (amount values) for all data rows — this sheet carries the structure only, amounts are in the cost-allocation sheet.
    - Auto-fits all columns.

#### Phase 9 — Format KostenverdelingLooncomponenten (Cost Allocation)
21. Activates the `KostenverdelingLooncomponenten` sheet and:
    - Deletes columns K through N.
    - Inserts a new header row 1: `A1 = "Import"`, `E1 = "Kostenplaats"`.
    - Inserts a second header row at row 2 with column labels: `PeriodeStart | PeriodeEind | Code | Waarde | Kostenplaats | Kostensoort`.
    - For every data row:
      - **PeriodeEind (F)** = copy of PeriodeStart (E) — both periods are the same date.
      - **Code (G)** = `"U2101"` (hardcoded Nmbrs cost-type code).
      - **Waarde (H)** = rounded to 2 decimal places, formatted as `#.00`.
      - **Kostensoort (J)** = first 5 characters of the cost type from column I.
      - **Kostenplaats (I)** = same value as the derived Kostensoort.
      - Column J is then cleared.
    - Deletes rows where column A is empty.
    - Auto-fits all columns.

#### Phase 10 — Save Nmbrs Export Files
22. Saves `LooncomponentenVar` sheet as a standalone workbook:  
    📄 **`<workbook folder>\LooncomponentenVar.xlsx`**  
    (extension may differ if `ExportExtention` named range is set, e.g. `.xlsb`)
23. Saves `KostenverdelingLooncomponenten` sheet as a standalone workbook:  
    📄 **`<workbook folder>\KostenverdelingLooncomponenten.xlsx`**
24. Closes both saved workbooks.
25. Deletes the `Temp` worksheet.

#### Phase 11 — Update Telling (Duplicate Detection)
26. Activates the **Telling sheet** and refreshes its pivot table (`pivTelling`).
27. Adds column E with the header `"<="` and the formula:
    ```
    =IF(AND(employee<>"", OR(employee=row_above, employee=row_below)), "<=", "")
    ```
    This marks employees appearing on two consecutive rows (i.e., they work at more than one store — their record appears twice).
28. Applies AutoFilter to column E.

#### Phase 12 — Final Status
29. Records completion timestamp and imported store count.
30. Updates **H8** formula: `=COUNTIF(E:E,"V")` (count of successfully imported stores).
31. Applies red conditional formatting to column E for `"X"` rows (failed stores).
32. Shows a message box:
    - If all stores imported: `"Imported: X stores."`
    - If some failed: `"Not all stores imported. Stores: X. Imported: Y."`
33. Returns focus to the Stores sheet.
34. Forces full recalculation and pivot refresh.

---

## 5. Output Files Summary

| File | Location | Format | Content | Used by |
|---|---|---|---|---|
| `HR-payroll.csv` | Same folder as the .xlsm | CSV (separator from I5) | All raw payroll rows from all stores, including manager column, multi-store annotations | Internal HR / reporting / audit trail |
| `LooncomponentenVar.xlsx` | Same folder as the .xlsm | Excel (.xlsx or .xlsb) | Variable salary component records per employee, formatted for Nmbrs import (no managers, no amounts in col H) | **Nmbrs payroll import — salary components** |
| `KostenverdelingLooncomponenten.xlsx` | Same folder as the .xlsm | Excel (.xlsx or .xlsb) | Cost-centre allocation per salary component, with period dates, cost code U2101, rounded amounts, cost type codes | **Nmbrs payroll import — cost allocation** |

---

## 6. Result Sheet — Column Structure

The Result sheet accumulates raw rows from the SQL stored procedure. After processing, it contains:

| Column | Content |
|---|---|
| A | Employee debtor number (may be remapped for multi-store employees) |
| B | Employee number |
| C | Department / cost centre code |
| D | Department name |
| E | Period start date |
| F–K | Payroll component data (amounts, codes, etc. from SQL) |
| L | Manager name (VLOOKUP from Managers sheet, added after extraction) |
| M | Multi-store annotation: `"Multi was (original_debtor)"` if debtor was remapped |

---

## 7. Nmbrs Import Files — Column Structure

### LooncomponentenVar.xlsx

| Row | Content |
|---|---|
| Row 1 | Header: `"Import"` in A1, `"Looncomponenten"` in E1 |
| Row 2+ | Employee salary component data (columns A–H; column H amounts cleared) |

### KostenverdelingLooncomponenten.xlsx

| Row | Content |
|---|---|
| Row 1 | Header: `"Import"` in A1, `"Kostenplaats"` in E1 |
| Row 2 | Column headers: PeriodeStart, PeriodeEind, Code, Waarde, Kostenplaats, Kostensoort |
| Row 3+ | Cost allocation per salary component per employee |

Column mapping in KostenverdelingLooncomponenten:

| Column | Header | Content |
|---|---|---|
| A | Employee debtor number | |
| B | Employee number | |
| C | Department code | |
| D | Department name | |
| E | PeriodeStart | Start of pay period |
| F | PeriodeEind | End of pay period (= copy of PeriodeStart) |
| G | Code | Always `"U2101"` (Nmbrs cost type code) |
| H | Waarde | Amount (rounded to 2 decimals) |
| I | Kostenplaats | First 5 chars of cost type |
| J | Kostensoort | Cleared (blank) |

---

## 8. Typical User Workflow

```
Step 1  Open the workbook
        → Today's date auto-set in G2
        → Workbook backup automatically created
        → Green circle on [Ping] button

Step 2  (First time only) Click [Load Store File]
        → Browse to the stores.dat file
        → Path saved in G3

Step 3  (First time only) Click [Load Standards File]
        → Browse to the Standards reference file
        → Path saved in G4

Step 4  Click [Reload Stores]
        → Store list populated from .dat file
        → Active stores marked "V" in column A

Step 5  Click [Ping All Stores]
        → Each store pinged over the network
        → Unreachable stores deactivated
        → Column D shows connectivity status

Step 6  (Optional) Adjust date in G2 if not running for today
        (Optional) Check/uncheck JustOneDay in I6

Step 7  (Optional) Click [Reload Standards] if Standards file changed

Step 8  Click [Run In All Stores]  ← MAIN ACTION
        → SQL data extracted from all connected stores
        → Multi-store employees corrected
        → Manager names added
        → 3 output files saved to disk
        → Nmbrs pivot updated
        → Summary message box shown
```

---

## 9. Automation / Unattended Mode

The workbook supports scheduled / unattended execution via Windows environment variables:

| Variable | Value | Effect |
|---|---|---|
| `DPEU_AUTORUN` | `1` | On open: automatically runs the full pipeline (ReloadStores → Ping → ReloadStandards → RunInAllStores) |
| `DPEU_AUTOCLOSE` | `1` | After completing the run: saves the workbook and quits Excel |

This allows the tool to be triggered from a Windows Task Scheduler job without user interaction.

---

## 10. Technical Notes

| Aspect | Detail |
|---|---|
| Database connection | Microsoft OLE DB Provider for SQL Server (`SQLOLEDB.1`) |
| Database user | `maint` (shared service account) |
| Database name | `DPEU` |
| Stored procedure | `DPEU.dbo.spDOM_ExtractPayrollNmbrsNL` |
| Store hostname format | `dp<country><storenum>.dpev.net` |
| Nmbrs cost code | `U2101` (hardcoded — Kostenpost for salary cost allocation) |
| Export format | `.xlsx` by default; configurable via named range `ExportExtention` |
| File encoding for .dat | `iso-8859-1` (Western European) |
| Sheet protection | All sheets are protected; macros unprotect before writing and re-protect after |
