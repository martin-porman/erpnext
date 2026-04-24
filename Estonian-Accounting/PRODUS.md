# PRODUS — ERPNext / Joosep / Trikato Accounting OS Integration Roadmap

**Created:** 2026-04-24  
**Author:** Investigation session based on Erp.txt task brief  
**Repo:** `martin-porman/erpnext` fork + Trikato Accounting OS  
**Status:** Phase 1 — Planning & Investigation  

---

## Phase Decision: ERPNext Direct Patching Deferred

> **Direct patches into the `martin-porman/erpnext` fork are not needed at this moment.**
>
> **Reason:** The integrations are mostly ready in terms of source material, but the project must proceed step by step. Before patching ERPNext directly, we need to complete Joosep source mapping, two-company validation, customer workflow design, migration planning, and Accounting OS comparison.
>
> **Decision:** ERPNext/Frappe direct patching is moved to **Phase 2**.
>
> Phase 2 starts only after:
> - Joosep field and workflow mapping is documented
> - A&T Ehitusjuhtimine and Lingua Estonica OÜ validation data is extracted
> - Trikato customer-management workflows are mapped
> - ERPNext gaps are identified with evidence
> - The full roadmap is split into executable work packages

---

## Table of Contents

1. [Current Project Status](#1-current-project-status)
2. [Source References](#2-source-references)
3. [Joosep Investigation Scope](#3-joosep-investigation-scope)
4. [Two-Company Validation Plan](#4-two-company-validation-plan)
5. [Full Accounting Workflow Map](#5-full-accounting-workflow-map)
6. [ERPNext / Frappe Integration Scope](#6-erpnext--frappe-integration-scope)
7. [Trikato Accounting Company Workflow](#7-trikato-accounting-company-workflow)
8. [E-Invoice / XML / SOAP Roadmap](#8-e-invoice--xml--soap-roadmap)
9. [X-Road / Maksuamet Roadmap](#9-x-road--maksuamet-roadmap)
10. [Joosep Migration Roadmap](#10-joosep-migration-roadmap)
11. [Independent Work Packages](#11-independent-work-packages)
12. [Decision Matrix](#12-decision-matrix)
13. [Execution Order](#13-execution-order)
14. [Risks, Unknowns, and Decisions Required from Martin](#14-risks-unknowns-and-decisions-required-from-martin)

---

## 1. Current Project Status

### What exists

| Asset | Location | Status |
|-------|----------|--------|
| `martin-porman/frappe` fork | GitHub + `/home/martin/dev/frappe` | Cloned at v16.13.0, Trikato branding applied, `et.po` locale present |
| `martin-porman/erpnext` fork | GitHub + `/home/martin/dev/erpnext` | Cloned at v16.13.0, `et.po` locale present |
| ERPNext v16.13.0 running | `frappe_docker` containers on port 8085 | Live, 10+ days uptime |
| Trikato Chart of Accounts | `Estonian-Accounting/Example/osaühing TRIKATO Kontoplaan 15.04.2026.xlsx` | 206 accounts, ready to import |
| KMD XML example | `Estonian-Accounting/Example/Lingua Estonica OÜ_KD-03-2026.xml` | Real KMD6 format confirmed |
| LHV CAMT.053 example | `Estonian-Accounting/Example/EE0377...xml` | 349 transactions, ISO 20022 |
| Joosep5 reference docs | `Estonian-Accounting/*.md` (5 documents, ~3700 lines total) | VAT, Ledger, Reports, Payments, PRG Index, Menu Map |
| Joosep source | Remote at `192.168.10.6:/home/martin/dev/Joosep/` | 177 PRG files decompiled, SQL schema documented |
| Trikato Baserow customers | `http://192.168.10.6:3001/database/7/table/806/` | 676 customers |
| `erpnext-mcp-server` | `/home/martin/dev/erpnext-mcp-server/` | Configured to `localhost:8085`, API keys set |

### What is NOT done

- ERPNext not configured for Estonian accounting (no CoA, no VAT templates, no dimensions)
- No custom Frappe app created for Estonian localization
- No Joosep data extracted for A&T Ehitusjuhtimine or Lingua Estonica OÜ
- No customer workflow designed in ERPNext or Baserow
- No e-invoice module
- No X-Road integration
- No data migration scripts

### What is postponed to Phase 2

- Direct patches to `martin-porman/erpnext` or `martin-porman/frappe`
- E-invoice XML generation module
- X-Road / EMTA submission module
- Joosep5 data migration scripts
- SEPA payment file generation
- TSD payroll tax declaration

---

## 2. Source References

### Local files

| Path | Content |
|------|---------|
| `/home/martin/dev/erpnext/Estonian-Accounting/` | All planning docs, XML examples, Excel CoA |
| `/home/martin/dev/erpnext/Estonian-Accounting/ESTONIAN_ACCOUNTING_ERPNEXT_REFERENCE.md` | Master reference — all sections |
| `/home/martin/dev/erpnext/Estonian-Accounting/VAT_AND_TAX_LOGIC.md` | 583 lines — VAT formulas, rates, dual-level codes |
| `/home/martin/dev/erpnext/Estonian-Accounting/LEDGER_AND_POSTING_LOGIC.md` | 871 lines — CoA, double-entry, journal rules |
| `/home/martin/dev/erpnext/Estonian-Accounting/REPORTS_AND_PERIOD_CLOSING.md` | 985 lines — reports, period closing, year-end |
| `/home/martin/dev/erpnext/Estonian-Accounting/PAYMENTS_AND_INTEGRATIONS.md` | 897 lines — banks, e-invoice, X-Road, SOAP |
| `/home/martin/dev/erpnext/Estonian-Accounting/PRG_INDEX.md` | 360 lines — all 177 PRG files indexed |
| `/home/martin/dev/erpnext/Estonian-Accounting/MENU_MAP.md` | 350 lines — menu → form → table → action |

### Remote (192.168.10.6)

| Path | Content |
|------|---------|
| `/home/martin/dev/Joosep/02-vfp-source/refox-decompiled/` | 1,761 decompiled VFP source files |
| `/home/martin/dev/Joosep/01-database/Joosep.mssql.md` | Full SQL Server schema |
| `/home/martin/dev/Joosep/03-screenshots/` | UI screenshots |
| `/home/martin/dev/Joosep/06-import-pipelines/` | Existing pipeline docs |
| SQL Express live | `192.168.10.12:1600` — Joosep5 production database |

---

## 3. Joosep Investigation Scope

### PRG files mapped to accounting areas

| Area | PRG File(s) | Status |
|------|------------|--------|
| Customers | `laoeksportimport_uus.prg` (KLIENTIMPORT), `klkontroll.prg`, `klkontroll2.prg` | Documented in LEDGER |
| Sales invoices | `earved.prg`, `arvete_import.prg`, `lisakontaktarve.prg` | Documented in PAYMENTS |
| Purchase invoices | `ostuarvete_import.prg` | Partially documented |
| E-invoices | `earved.prg` (52.5K), `earvekeskus_soap.prg`, `envoice_soap.prg`, `telemaxml.prg` | Documented in PAYMENTS |
| Payments | `swed_gateway.prg`, `lhv_connect.prg`, `seb_gw.prg`, `kaardimakse.prg`, `sepa_xml.prg` | Documented in PAYMENTS |
| Bank statements | `swed_gateway.prg` (CAMT.053), `lhv_connect.prg` | Documented in PAYMENTS |
| Accounting entries | `kandesalv.prg`, `arufunk.prg`, `bilanss.prg`, `kasum2.prg` | Documented in LEDGER |
| VAT / KMD | `VAT_AND_TAX_LOGIC.md` (full), `xtee_class.prg` (X-Road submission) | Documented in VAT |
| TSD (payroll tax) | `tsd_xml.prg` (15.8K, 8 procedures) | **Not yet detailed** |
| Annual report | `bilanss.prg`, `kasum2.prg`, `REPORTS_AND_PERIOD_CLOSING.md` | Documented in REPORTS |
| Maksuamet / X-Road | `xtee_class.prg` (36.8K — largest integration file), `maksuamet_xtee.prg` (SCX form) | Structure known, XML not yet mapped |
| SOAP integrations | `wwsoap.prg`, `jsp_wwsoap.prg`, `earvekeskus_soap.prg`, `envoice_soap.prg` | Documented in PAYMENTS |
| Data export/import | `laoeksportimport_uus.prg`, `laoeksportimport.prg` | Partially documented |
| Company-specific data | `firma` table, `firmnr`, `firmknr`, `firmkmnr` fields | Documented in LEDGER |

### Not found in current source index

- `maksuamet_xtee.prg` — referenced as SCX (compiled form), source not in PRG index. The logic lives in `xtee_class.prg`.
- TSD payroll field-level mapping — `tsd_xml.prg` structure known but field-to-ERPNext mapping not done.
- Inforegister API field mapping — `inforegister_api.prg` listed but not documented.
- Erply integration mapping — `erply_import.prg` listed, not detailed for migration purposes.

---

## 4. Two-Company Validation Plan

### Companies

| Company | Purpose |
|---------|---------|
| **A&T Ehitusjuhtimine** | Construction services — complex inventory, projects, subcontractors |
| **Lingua Estonica OÜ** | Service company — simpler, clean invoicing, existing KMD example available |

### Data available now

| Asset | Company | Location |
|-------|---------|----------|
| Opening balances XML | A&T Ehitusjuhtimine | `Example/2024 algsaldod ehitusjutimine.xml` |
| KMD declaration | Lingua Estonica OÜ | `Example/Lingua Estonica OÜ_KD-03-2026.xml` |
| Balance sheet PDF | Lingua Estonica OÜ | `Example/Lingua Estonica OÜ Bilanss 03.26.pdf` |
| P&L PDF | Lingua Estonica OÜ | `Example/Lingua Estonica OÜ Kasumiaruanne 03.26.pdf` |
| Day book PDF | Lingua Estonica OÜ | `Example/Lingua Estonica OÜ Päevaraamat 03.26.pdf` |

### Joosep data lag

Joosep database is **~3 months behind current** state. This is intentional for comparison:
- Joosep data = historical reference baseline
- Current accounting = forward-moving system
- Mismatch between Joosep and new system = debugging signal

### Validation checklist (per company)

- [ ] Extract all documents (invoices, payments, entries) for Q1 2026
- [ ] Extract chart of accounts with opening balances
- [ ] Extract bank statement transactions (matched vs unmatched)
- [ ] Extract KMD declaration data
- [ ] Map Joosep document types to ERPNext DocTypes
- [ ] Compare GL totals (debit = credit per period)
- [ ] Compare VAT amounts (KMD rows vs ledger)
- [ ] Compare customer/vendor balances (aging)
- [ ] Identify mismatches → log as bugs or design decisions

### Extraction method

Data is on SQL Express at `192.168.10.12:1600`. Extraction via:
- Direct SQL queries to `dokument`, `kanne_key`, `liikumin`, `klient`, `tasumine` tables
- Or via existing Joosep export functions (`laoeksportimport_uus.prg` KLIENTIMPORT etc.)

---

## 5. Full Accounting Workflow Map

```
Customer (klient table)
  → Documents (dokument / dokument_key)
      → Sales Invoice (tyyp=1)         → ERPNext: Sales Invoice
      → Purchase Invoice (tyyp=4)      → ERPNext: Purchase Invoice
      → Advance Invoice (tyyp=6)       → ERPNext: Sales Invoice (is_advance=1)
      → Credit Note (tyyp=7)           → ERPNext: Sales Invoice (is_return=1)
      → Journal Entry (tyyp=5)         → ERPNext: Journal Entry
      → E-Invoice (tyyp=34)            → ERPNext: Custom (e-invoice module)
  → Bank Statements (CAMT.053 / swed_gateway / lhv_connect)
      → Import → bank_transaction      → ERPNext: Bank Transaction (native CAMT.053)
  → Payments (tasumine table)
      → Customer receipt (tyyp=2)      → ERPNext: Payment Entry
      → Supplier payment (tyyp=4+pay)  → ERPNext: Payment Entry
      → SEPA batch (sepa_xml.prg)      → ERPNext: Payment Order
  → Accounting Entries (kanne / kanne_key)
      → Double-entry per document      → ERPNext: GL Entry (auto-generated)
      → Monthly aggregation (kuukande) → ERPNext: Period GL summary
  → VAT / Tax Declarations
      → KMD (käibedeklaratsioon)       → ERPNext: Custom report → XML → X-Road
      → TSD (töötasude deklaratsioon)  → Custom (Phase 2)
  → Reports
      → Bilanss (Balance Sheet)        → ERPNext: Balance Sheet (native)
      → Kasumiaruanne (P&L)            → ERPNext: Profit and Loss (native)
      → Pearaamat (General Ledger)     → ERPNext: General Ledger (native)
      → Proovibilanss (Trial Balance)  → ERPNext: Trial Balance (native)
      → KMD annex                      → ERPNext: Custom report
  → Validation against Joosep output  → Comparison checklist
```

### Per-workflow classification

| Workflow | Source | ERPNext Native? | Needs Patch? | Wrapper? | Migration? | Phase |
|----------|--------|-----------------|-------------|---------|-----------|-------|
| Chart of accounts | Trikato Excel (206 accounts) | Config only | No | No | Import JSON | Phase 1 |
| VAT templates (22/9/5/0/exempt) | VAT_AND_TAX_LOGIC.md | Config only | No | No | Setup | Phase 1 |
| Cost center dimensions | LEDGER doc | Config only | No | No | Setup | Phase 1 |
| Customer master | Baserow table 806 | Config only | No | No | Import | Phase 1 |
| Sales invoice | tyyp=1, earved.prg | Native | No | No | Migrate | Phase 1 |
| Purchase invoice | tyyp=4 | Native | No | No | Migrate | Phase 1 |
| Payment entry | tyyp=2, tasumine | Native | No | No | Migrate | Phase 1 |
| Journal entry | tyyp=5, kandesalv.prg | Native | No | No | Migrate | Phase 1 |
| Bank import CAMT.053 | swed/lhv gateway | Native | No | No | Config | Phase 1 |
| Payment reconciliation | tasumine + viide | Partial | Maybe | Yes | Config | Phase 1 |
| Period closing | peralgus, bilsalv | Partial | Maybe | No | Config | Phase 1 |
| Balance Sheet / P&L | bilanss.prg, kasum2.prg | Native | No | No | Config | Phase 1 |
| KMD declaration XML | xtee_class.prg | No | No | Yes (custom report) | Build | Phase 2 |
| E-invoice v1.11 XML | earved.prg (52.5K) | No | No | Yes (custom module) | Build | Phase 2 |
| OMNIVA/Envoice SOAP | earvekeskus_soap.prg | No | No | Yes | Build | Phase 2 |
| X-Road submission | xtee_class.prg (36.8K) | No | No | Yes | Build | Phase 2 |
| TSD payroll declaration | tsd_xml.prg | No | No | Yes | Build | Phase 2 |
| SEPA payment file | sepa_xml.prg | Partial | No | Yes | Build | Phase 2 |
| Joosep data migration | All tables | No | No | Scripts | Build | Phase 1 |
| Customer management | Baserow table 806 | Partial | Maybe | Yes | Design | Phase 1 |
| Accountant workflow | Trikato process | No | No | Yes (custom) | Design | Phase 1 |
| Reg code validation | leiaiban.prg | No | Server script | No | Build | Phase 1 |
| IBAN validation | leiaiban.prg (KONTROLLIIBAN) | No | Server script | No | Build | Phase 1 |

---

## 6. ERPNext / Frappe Integration Scope

### Configure only (no code, Phase 1)

1. **Chart of Accounts** — import 206 accounts from Trikato Excel as ERPNext CoA JSON
2. **VAT Tax Templates** — 5 templates: KM 22%, KM 9%, KM 5%, KM 0%, Vaba (exempt)
3. **Accounting Dimensions** — 4 dimensions: Osakond (Cost Center), Objekt, Projekt, Lisatunnus4
4. **Fiscal Year** — calendar year Jan–Dec
5. **Naming Series** — ARV-.YYYY.-, OST-.YYYY.-, KANNE-.YYYY.-, ETTEMAKS-.YYYY.-
6. **Company setup** — Trikato OÜ + multi-company for managed clients

### Server scripts / custom fields (Phase 1)

- Estonian reg code format validation on Customer/Supplier
- IBAN format validation (EE + 20 chars)
- VAT number format validation (EExxxxxxxxxx)
- Auto-rounding line on invoice save (> 0.005 mismatch → "ümardus" line)

### Custom DocTypes needed (Phase 2)

- `Estonian KMD Declaration` — header + body fields + annex lines
- `Estonian E-Invoice` — maps to earved.prg v1.11 schema
- `X-Road Submission Log` — audit trail for EMTA submissions
- `Joosep Import Log` — migration tracking

### ERPNext already covers (confirmed)

- Double-entry GL
- CAMT.053 ISO 20022 bank import
- Multi-currency with exchange rates
- Payment reconciliation tool
- Trial Balance, General Ledger, Balance Sheet, P&L (all native)
- Multi-company
- Period closing voucher
- Fiscal year management

---

## 7. Trikato Accounting Company Workflow

Trikato is an accounting services company managing **676 customers** (Baserow table 806, database 7).

### Role model

```
Administrator / CEO
  → Full visibility: all customers, all accountants, all financials
  → Can assign / reassign customers to accountants
  → Tracks customer profitability, onboarding status, risk
  → Views invoice/payment status across all customers
  → Sees accountant workload and completion rates

Accountant
  → Sees only assigned customers
  → Monthly workflow per customer:
      1. Collect bank statements
      2. Collect invoices (purchase + sales)
      3. Reconcile payments
      4. Post accounting entries
      5. Submit KMD declaration
      6. Submit TSD (if payroll)
      7. Generate reports (bilanss, kasumiaruanne, pearaamat)
  → Tracks: missing documents, bank statement status, reconciliation status
  → Tracks: KMD submitted, TSD submitted, annual report submitted

Customer (Phase 2)
  → Portal / communication layer — not Phase 1
```

### Baserow → ERPNext mapping

| Baserow field (table 806) | ERPNext equivalent |
|--------------------------|-------------------|
| Customer name | Customer.customer_name |
| Registrikood | Customer.tax_id |
| KMKR | Customer.vat_id (custom) |
| Assigned accountant | Custom field: Customer.accountant (link to User) |
| Status | Custom field: Customer.status |
| Accounting system (Merit/Joosep) | Custom field: Customer.accounting_system |
| Contract type / fee | Custom field or Subscription |

### Monthly accountant task model (per customer)

| Task | Trigger | Status tracked |
|------|---------|---------------|
| Collect bank statement | Month end | Received / Missing |
| Collect sales invoices | Month end | Count / Missing |
| Collect purchase invoices | Month end | Count / Missing |
| Reconcile bank vs invoices | After receipt | % matched |
| Post journal entries | After reconciliation | Posted / Draft |
| KMD declaration | 20th of following month | Submitted / Pending |
| TSD declaration | 10th of following month | Submitted / Pending (if payroll) |
| Monthly report package | After KMD | Sent / Not sent |

This workflow can be tracked via ERPNext **ToDo**, **Task**, or a custom **Accounting Period Checklist** DocType per customer per month.

---

## 8. E-Invoice / XML / SOAP Roadmap

**Status: Phase 2 — Do not implement yet. Map only.**

### Source files

| File | Size | Role |
|------|------|------|
| `earved.prg` | 52.5K | XML generation for e-invoice v1.0 / 1.1 / 1.11 |
| `telemaxml.prg` | 32.2K | Telema EDI / EMTA XML format |
| `earvekeskus_soap.prg` | 3.8K | OMNIVA Arvekeskus SOAP client |
| `envoice_soap.prg` | 3.7K | Envoice SOAP client |

### E-invoice v1.11 root structure (confirmed from source)

```xml
<E_Invoice xmlns:xsi="..." xsi:noNamespaceSchemaLocation="e-invoice_ver1.11.xsd">
  <Header>
    <Date /><FileId /><Version>1.11</Version><AppId>EARVE</AppId>
  </Header>
  <Invoice invoiceId="{dok_nr}" regNumber="{kl_regnr}">
    <SellerParty><Name /><RegNumber /><VATRegNumber /><AccountInfo><IBAN /><BIC /></AccountInfo></SellerParty>
    <BuyerParty><Name /><RegNumber /><VATRegNumber /></BuyerParty>
    <InvoiceInformation><Type /><InvoiceNumber /><InvoiceDate /><DueDate /><PaymentReferenceNumber /></InvoiceInformation>
    <InvoiceSumGroup>
      <InvoiceSum /><VAT><SumBeforeVAT /><VATRate /><VATSum /></VAT><TotalSum />
    </InvoiceSumGroup>
    <InvoiceItem>...</InvoiceItem>
    <PaymentInfo><Currency /><PaymentRefId /><PayToAccount /><PayToName /></PaymentInfo>
  </Invoice>
  <Footer><TotalNumberInvoices /><TotalAmount /></Footer>
</E_Invoice>
```

### Field mapping (Joosep → ERPNext → XML)

| XML Tag | Joosep source | ERPNext source |
|---------|-------------|----------------|
| `InvoiceNumber` | `dok_nr` | `Sales Invoice.name` |
| `InvoiceDate` | `kuupaev` | `Sales Invoice.posting_date` |
| `DueDate` | `tasupaev` | `Sales Invoice.due_date` |
| `SellerParty.Name` | `firma.firmnimi` | `Company.company_name` |
| `SellerParty.RegNumber` | `firma.firmnr` | `Company.tax_id` |
| `SellerParty.VATRegNumber` | `firma.firmkmnr` | `Company.vat_id` |
| `SellerParty.IBAN` | `omapank_key.oma_iban` | `Company Bank Account.iban` |
| `BuyerParty.Name` | `klient.kl_nimi` | `Customer.customer_name` |
| `BuyerParty.RegNumber` | `klient.kl_regnr` | `Customer.tax_id` |
| `BuyerParty.VATRegNumber` | `klient.kl_kmnr` | `Customer.vat_id` |
| `InvoiceSum` | `SUM(arveread.summa)` | `Sales Invoice.net_total` |
| `VATRate` | `kmarray(i,1)` | From tax template |
| `VATSum` | `kmarray(i,4)` | `Sales Invoice.total_taxes_and_charges` |
| `TotalSum` | `dokument.kokkusum` | `Sales Invoice.grand_total` |
| `PayToAccount` | `kl_iban` | `Customer.bank_account.iban` |
| `PaymentRefId` | `dok_viide` | `Sales Invoice.payment_reference` |

### SOAP operators

| Operator | Endpoint | Protocol |
|----------|---------|---------|
| OMNIVA Arvekeskus | `arvetekeskus.eu` | SOAP — `EInvoiceRequest`, `BuyInvoiceRequest` |
| Envoice | `envoice.ee` | SOAP — parallel API |

### Open questions

- **Which operator does Trikato use?** OMNIVA or Envoice? (Decision needed from Martin)
- **Are e-invoices sent for all customers or only those who request it?**
- **Incoming e-invoice parsing** — is earved.prg used for receiving? (Confirmed: it parses both directions)

---

## 9. X-Road / Maksuamet Roadmap

**Status: Phase 2 — Map only, do not implement.**

### Source files

| File | Size | Role |
|------|------|------|
| `xtee_class.prg` | 36.8K | X-Tee security layer — largest integration file |
| `tsd_xml.prg` | 15.8K | TSD XML format (payroll tax declaration) |
| `maksuamet_xtee.scx` | compiled | UI form for EMTA submission (not decompilable) |

### Key procedures in `xtee_class.prg`

| Procedure | Purpose |
|-----------|---------|
| `KINNITATSD` | Confirm/submit TSD declaration |
| `KONTROLLISTAATUS` | Check submission status |
| `KONTROLLIVIGU` | Validate before submission |
| `LISABODY` | Add SOAP body to X-Road envelope |

### X-Road requirements

- **Security server** — Must be registered with RIA (Estonian Information System Authority). Either own security server or third-party relay (e.g., Cybernetica, Helmes).
- **Certificate** — PFX/P12 certificate from SK ID Solutions or similar
- **Service registration** — Register as a service consumer for EMTA services

### Open question (blocks Phase 2)

> **Does Trikato have access to an X-Road security server, or will a relay service be used?**  
> This is the single biggest Phase 2 blocker. Without an answer, X-Road cannot be planned.

### KMD declaration XML (confirmed format)

```xml
<vatDeclaration>
  <taxPayerRegCode>...</taxPayerRegCode>
  <year>2026</year><month>3</month>
  <declarationType>1</declarationType><version>KMD6</version>
  <declarationBody>
    <noSales>false</noSales><noPurchases>false</noPurchases>
    <transactions24>...</transactions24>
    <inputVatTotal>...</inputVatTotal>
    <euAcquisitionsGoodsAndServicesTotal>...</euAcquisitionsGoodsAndServicesTotal>
  </declarationBody>
  <salesAnnex>...</salesAnnex>
  <purchasesAnnex>
    <purchaseLine>
      <sellerRegCode /><invoiceNumber /><invoiceDate />
      <invoiceSumVat /><vatInPeriod />
    </purchaseLine>
  </purchasesAnnex>
</vatDeclaration>
```

---

## 10. Joosep Migration Roadmap

**Status: Phase 1 — Extract and document. Do not migrate until gap analysis complete.**

### Source

- SQL Express at `192.168.10.12:1600`
- Database: `RP` (Joosep naming convention)
- Schema documented at: `/home/martin/dev/Joosep/01-database/Joosep.mssql.md`

### Tables to extract per company

| Table | Content | ERPNext target |
|-------|---------|----------------|
| `konto` | Chart of accounts | Account |
| `klient` | Customers + suppliers | Customer / Supplier |
| `dokument` / `dokument_key` | Document headers | Sales Invoice / Purchase Invoice / Journal Entry |
| `liikumin` / `liikumin_key` | Document line items | Invoice Item lines |
| `kanne` / `kanne_key` | GL entries | GL Entry |
| `tasumine` | Payments | Payment Entry |
| `algsaldo` | Opening balances | Opening Journal Entry |
| `peralgus` | Period definitions | Accounting Period |
| `kuukande` | Monthly aggregates | (reference only) |

### Migration risk areas

| Risk | Detail |
|------|--------|
| Account code mapping | Joosep uses 6-digit, ERPNext uses name-based. Must map 1:1. |
| Document numbering | Joosep dok_nr preserved as naming_series prefix |
| Open invoices | Unpaid invoices at migration cutoff must carry forward correctly |
| VAT tax code mapping | Joosep `lmaks.maksnr` → ERPNext Tax Category |
| Multi-currency entries | `summa` + `summaval` + `val_kurss` — all must migrate |
| Rounding lines | `ümardus` auto-lines must be preserved in migration |
| Company isolation | Each Joosep company = separate ERPNext company |

---

## 11. Independent Work Packages

### Stack 1 — Joosep Source Understanding

**Goal:** Document accounting logic, identify migration-critical tables and fields.

Tasks:
- [ ] Read `xtee_class.prg` full source → document X-Road call structure
- [ ] Read `tsd_xml.prg` → document TSD field-to-ERPNext mapping
- [ ] Map `klient` table fields → ERPNext Customer fields
- [ ] Map `dokument` + `kanne_key` → ERPNext Sales Invoice + GL Entry
- [ ] Document `tasumine` → Payment Entry mapping
- [ ] Document `peralgus` → Accounting Period mapping
- [ ] Document `algsaldo` → Opening Balance migration

**Input:** PRG_INDEX.md, LEDGER_AND_POSTING_LOGIC.md, MSSQL schema  
**Output:** `JOOSEP_FIELD_MAP.md`  
**Blocker:** Remote SSH access to `192.168.10.6`

---

### Stack 2 — Two-Company Validation Dataset

**Goal:** Extract real data for A&T Ehitusjuhtimine and Lingua Estonica OÜ.

Tasks:
- [ ] SQL query: all documents (Q1 2026) for both companies
- [ ] SQL query: GL entries (kanne_key) for Q1 2026
- [ ] SQL query: payment records (tasumine) for Q1 2026
- [ ] SQL query: opening balances (algsaldo) at 2026-01-01
- [ ] SQL query: bank transactions (if stored in Joosep)
- [ ] Compare Lingua Estonica KMD XML against GL totals
- [ ] Build comparison checklist (Joosep output vs expected ERPNext output)

**Input:** SQL Express at `192.168.10.12:1600`  
**Output:** `VALIDATION_DATASET.md` + SQL export files  
**Blocker:** SQL access to `192.168.10.12:1600`

---

### Stack 3 — ERPNext / Frappe Gap Analysis

**Goal:** Confirm what ERPNext covers natively and what gaps exist.

Tasks:
- [ ] Configure ERPNext with Trikato CoA (import 206 accounts)
- [ ] Set up 5 VAT tax templates
- [ ] Set up 4 accounting dimensions
- [ ] Test sales invoice posting → check GL entries
- [ ] Test CAMT.053 import with LHV example file
- [ ] Test payment reconciliation
- [ ] Test period closing
- [ ] Test Balance Sheet and P&L reports
- [ ] Document gaps found during testing

**Input:** Running ERPNext at port 8085, erpnext-mcp-server  
**Output:** `GAP_ANALYSIS.md`  
**Blocker:** None — can start immediately

---

### Stack 4 — Trikato Customer Management

**Goal:** Design admin + accountant workflow for 676 customers.

Tasks:
- [ ] Read Baserow table 806 full schema (all fields)
- [ ] Define ERPNext Customer custom fields needed
- [ ] Design accountant assignment model (1 accountant → N customers)
- [ ] Design monthly task checklist per customer
- [ ] Define status model: Onboarding / Active / Suspended / Churned
- [ ] Define performance metrics: invoices processed, KMDs submitted, late items
- [ ] Design CEO dashboard requirements
- [ ] Decide: ERPNext custom DocTypes vs Baserow vs hybrid

**Input:** Baserow `http://192.168.10.6:3001/database/7/table/806/`  
**Output:** `CUSTOMER_WORKFLOW.md`  
**Blocker:** Baserow API access

---

### Stack 5 — E-Invoice / SOAP / X-Road

**Goal:** Document all requirements. No implementation.

Tasks:
- [ ] Map `earved.prg` procedure by procedure → document all XML fields
- [ ] Document OMNIVA Arvekeskus SOAP request/response format
- [ ] Document Envoice SOAP request/response format
- [ ] Map `xtee_class.prg` X-Road envelope format
- [ ] Document RIA certificate requirements
- [ ] Define: which operator does Trikato use?
- [ ] Document incoming e-invoice parsing requirements

**Input:** PRG_INDEX.md, PAYMENTS_AND_INTEGRATIONS.md, remote source  
**Output:** `EINVOICE_SPEC.md`, `XROAD_SPEC.md`  
**Blocker:** Decision from Martin on OMNIVA vs Envoice

---

### Stack 6 — Accounting OS Wrapper Decision

**Goal:** For each feature, determine where it lives.

For each major workflow, classify as:
- A) Native ERPNext (config only)
- B) ERPNext server script
- C) ERPNext custom DocType
- D) External Python service (Accounting OS wrapper)
- E) Joosep migration script
- F) Human workflow / admin process
- G) Deferred to Phase 2

**Input:** Results from Stacks 1–5  
**Output:** Updated decision matrix in this document  
**Blocker:** Depends on Stacks 3 and 4

---

## 12. Decision Matrix

| Workflow | Source Evidence | ERPNext Native? | Needs Patch? | Wrapper? | Migration? | Phase |
|----------|----------------|-----------------|-------------|---------|-----------|-------|
| Chart of Accounts | Trikato Excel 206 accounts | Config | No | No | Import | **1** |
| VAT templates | VAT_AND_TAX_LOGIC.md | Config | No | No | Setup | **1** |
| Accounting dimensions | LEDGER doc | Config | No | No | Setup | **1** |
| Customer master | Baserow table 806 | Config + custom fields | Minor | No | Import | **1** |
| Sales / Purchase Invoice | tyyp 1,4 in dokument | Native | No | No | Migrate | **1** |
| Journal Entry | tyyp=5, kandesalv.prg | Native | No | No | Migrate | **1** |
| Payment Entry | tasumine table | Native | No | No | Migrate | **1** |
| Bank CAMT.053 import | swed/lhv gateway | **Native** | No | No | Config | **1** |
| Payment reconciliation | tasumine + viide | Partial | No | Server script | Config | **1** |
| Period closing | peralgus, bilsalv | Native | No | No | Config | **1** |
| Balance Sheet / P&L | bilanss.prg → native | **Native** | No | No | Config | **1** |
| Reg/IBAN validation | leiaiban.prg | No | Server script | No | Build | **1** |
| KMD declaration XML | xtee_class.prg | No | No | Custom report | Build | **2** |
| E-invoice v1.11 XML | earved.prg (52.5K) | No | No | Custom module | Build | **2** |
| OMNIVA/Envoice SOAP | earvekeskus/envoice | No | No | Python service | Build | **2** |
| X-Road submission | xtee_class.prg (36.8K) | No | No | Python service | Build | **2** |
| TSD payroll declaration | tsd_xml.prg | No | No | Custom module | Build | **2** |
| SEPA payment file | sepa_xml.prg | Partial | No | Python service | Build | **2** |
| Joosep data migration | All SQL tables | No | No | Python scripts | Build | **1** |
| Accountant workflow UI | Trikato process | No | Custom DocTypes | Partial | Design | **1** |
| CEO dashboard | Trikato process | No | Custom | No | Design | **1** |
| Intrastat | x_intrastat.scx | No | No | Custom | Build | **2** |
| Annual report EMTA | bilanss.prg + X-Road | Partial | No | Custom | Build | **2** |

---

## 13. Execution Order

```
Phase 1 — Foundation (start now)

  [1] Stack 3: ERPNext Gap Analysis
      → Configure CoA, VAT, dimensions on running ERPNext
      → Test native features with real files
      → Produces: GAP_ANALYSIS.md

  [2] Stack 4: Trikato Customer Management
      → Read Baserow table 806
      → Design workflows and roles
      → Produces: CUSTOMER_WORKFLOW.md

  [3] Stack 2: Two-Company Validation Dataset
      → Requires: SQL access to 192.168.10.12:1600
      → Extract Q1 2026 data for A&T + Lingua Estonica
      → Produces: VALIDATION_DATASET.md + SQL exports

  [4] Stack 1: Joosep Source Understanding
      → Deeper field-level mapping for migration
      → Requires: SSH to 192.168.10.6
      → Produces: JOOSEP_FIELD_MAP.md

  [5] Stack 5: E-Invoice / X-Road Spec
      → Document only, no implementation
      → Requires: Decision on OMNIVA vs Envoice
      → Produces: EINVOICE_SPEC.md, XROAD_SPEC.md

  [6] Stack 6: Wrapper Decision
      → Synthesize findings from 1–5
      → Finalize decision matrix
      → Produces: Updated PRODUS.md

Phase 2 — Implementation (after Phase 1 complete)

  [7] ERPNext Estonian localization app (bench new-app)
  [8] KMD report + XML export
  [9] E-invoice module
  [10] X-Road integration
  [11] Joosep data migration scripts
  [12] TSD / payroll declaration
  [13] Customer portal (if needed)
```

---

## 14. Risks, Unknowns, and Decisions Required from Martin

| # | Risk / Unknown | Severity | Decision needed |
|---|----------------|---------|----------------|
| 1 | **X-Road security server** — Does Trikato have one? | Critical (blocks Phase 2 KMD/X-Road) | Yes: own server, relay service, or Envoice handles KMD? |
| 2 | **E-invoice operator** — OMNIVA or Envoice? | High | Choose one, or both? |
| 3 | **SQL access to 192.168.10.12:1600** — Can this be queried directly? | High (blocks Stack 1+2) | Confirm SSH/SQL access method |
| 4 | **Migration scope** — Is Joosep data migration required for all 676 Trikato clients, or only for Trikato's own books? | High | Clarify scope |
| 5 | **Payroll / TSD** — Is TSD declaration in scope for Phase 1? | Medium | Confirm yes/no |
| 6 | **Localization app vs fork patches** — Confirmed: custom app preferred, but which repo hosts it? | Medium | New repo `martin-porman/erpnext-ee`? Or sub-folder in erpnext fork? |
| 7 | **Accounting system per customer** — Trikato uses both Merit and Joosep for clients. Is Merit integration in scope? | Medium | Confirm Merit scope |
| 8 | **Customer portal** — Is a customer-facing portal in Phase 1 scope? | Low | Confirm no for now |
| 9 | **Intrastat** — Is EU trade reporting needed? | Low | Confirm scope |
| 10 | **Lingua Estonica OÜ KMD** uses `declarationType=1` and has `noSales: true`. Is this company type representative of typical client? | Low | Confirm validation fixture is representative |

---

## Files Inspected

| File | Lines | Used for |
|------|-------|---------|
| `ESTONIAN_ACCOUNTING_ERPNEXT_REFERENCE.md` | ~400 | Master index and CoA |
| `VAT_AND_TAX_LOGIC.md` | 583 | VAT formulas, rates, field mapping |
| `LEDGER_AND_POSTING_LOGIC.md` | 871 | Double-entry rules, tables, document types |
| `REPORTS_AND_PERIOD_CLOSING.md` | 985 | Reports, period close, year-end |
| `PAYMENTS_AND_INTEGRATIONS.md` | 897 | Bank, e-invoice, X-Road, SOAP |
| `PRG_INDEX.md` | 360 | All 177 PRG files |
| `MENU_MAP.md` | 350 | Menu to form to table mapping |
| `Example/osaühing TRIKATO Kontoplaan.xlsx` | 206 accounts | Chart of accounts |
| `Example/Lingua Estonica OÜ_KD-03-2026.xml` | KMD6 | VAT declaration format |
| `Example/EE0377...xml` | 349 txns | CAMT.053 bank statement |
| `Example/2024 algsaldod ehitusjutimine.xml` | XFRX format | Opening balances format |
| `Erp.txt` | 323 lines | Task brief |

---

## Recommended Next Execution Step

**Start Stack 3 immediately** — it has no blockers and produces the most concrete evidence:

```bash
# Configure ERPNext at port 8085 via erpnext-mcp-server
# Step 1: Import Trikato Chart of Accounts
# Step 2: Create 5 VAT tax templates
# Step 3: Import LHV CAMT.053 bank statement
# Step 4: Create a test sales invoice, verify GL entries
# Step 5: Run Balance Sheet report
# Document what works natively and what fails
```

All other stacks require either remote access (192.168.10.6/192.168.10.12) or decisions from Martin.
