<p align="center">
  <img src="img/showcase-header.svg" alt="Warehouse Manager — dispatch, returns and recovered stock" width="100%">
</p>

<p align="center">
  <strong>From dispatched replacement to recovered stock—keep every unit accounted for.</strong>
</p>

<p align="center">
  <img src="img/badges/python.svg" alt="Python" height="28">
  <img src="img/badges/django.svg" alt="Django" height="28">
  <img src="img/badges/htmx.svg" alt="HTMX" height="28">
  <img src="img/badges/tailwind.svg" alt="Tailwind CSS" height="28">
  <img src="img/badges/sqlite.svg" alt="SQLite" height="28">
</p>

<p align="center">
  <a href="#the-problem">Problem</a> ·
  <a href="#the-solution">Solution</a> ·
  <a href="#app-walkthrough">Walkthrough</a> ·
  <a href="#application-architecture">Architecture</a> ·
  <a href="#data-architecture">Data model</a> ·
  <a href="#engineering-decisions">Engineering</a>
</p>

## The problem

When replacement equipment is dispatched for maintenance, the removed units are expected back. Manual follow-up made it difficult to answer three basic questions: **Has this unit returned? Where was it stored? Has it been handed over again?** Warehouse operators needed a traceable record of individual units; managers needed visibility into outstanding returns and recovered stock.

## The solution

**Warehouse Manager connects the original dispatch, each unit’s return, its warehouse location, and its next handover.** Staff can follow up on missing returns, find recovered units, and document stock issues with numbered PDF vouchers.

<p align="center">
  <a href="img/dashboard-overview.png"><img src="img/dashboard-overview.png" alt="Dashboard showing outstanding returns, current recovered stock, stock movements, and recovery progress" width="720"></a>
</p>

*Return progress, recovered stock, and recorded movements in one operational view.*

## App walkthrough

Follow a **Touch Panel** from dispatch review to receipt, recovered stock, and a documented handover. Select any image to open it at full size; expand the supporting views for more detail.

[01 · Track returns](#01--track-outstanding-returns) → [02 · Receive units](#02--receive-and-locate-units) → [03 · Issue stock](#03--issue-stock-and-manage-paperwork)

### 01 · Track outstanding returns

Filter **Expéditions** for outstanding returns. Progress indicators and dispatch information help staff identify what needs follow-up before opening an individual unit record.

<p align="center">
  <a href="img/dispatch-unit-status.png"><img src="img/dispatch-unit-status.png" alt="Dispatch list filtered for outstanding returns, with progress indicators and follow-up information" width="720"></a>
</p>

*Dispatch-level progress turns an expected return into a specific follow-up task.*

### 02 · Receive and locate units

Open the dispatch and select the unit’s return-status control. Equipment, part, and unit details establish which item is being received.

<p align="center">
  <a href="img/return-confirmation0.png"><img src="img/return-confirmation0.png" alt="Dispatch detail showing the Touch Panel unit and its return-status control" width="720"></a>
</p>

The receipt form records location, condition, and optional comments or photos. Here, the Touch Panel is recorded at **ETAGE2** as **Défectueuse – réparable** (defective, repairable).

<p align="center">
  <a href="img/return-confirmation1.png"><img src="img/return-confirmation1.png" alt="Receipt form recording the Touch Panel’s location, condition, comment, and optional photos" width="440"></a>
</p>

*A confirmed return connects the unit’s condition and warehouse location to its original dispatch.*

<details>
<summary><strong>See the returned part in recovered stock</strong></summary>

The **Stock** search makes the returned part accessible for later selection.

<p align="center">
  <a href="img/recovered-stock.png"><img src="img/recovered-stock.png" alt="Recovered stock filtered for Touch Panel, showing one returned unit" width="720"></a>
</p>

</details>

### 03 · Issue stock and manage paperwork

Select an available unit and review its handover before confirmation. The preview combines recipient, collector, issue reason, original dispatch, condition, and location.

<p align="center">
  <a href="img/stock-issue-preview.png"><img src="img/stock-issue-preview.png" alt="Stock issue preview with handover details and the selected Touch Panel unit" width="720"></a>
</p>

*Confirmation removes the unit from available stock and creates a numbered voucher linked to its original dispatch record.*

<details>
<summary><strong>Explore the handover record and generated voucher</strong></summary>

**The handover record.** Voucher details keep the issued unit and handover participants together, with an original PDF download and a place to attach the signed copy later.

<p align="center">
  <a href="img/voucher-detail.png"><img src="img/voucher-detail.png" alt="Voucher detail with handover information, issued unit, PDF download, and signed copy awaiting upload" width="720"></a>
</p>

**The printable document.** The generated voucher carries the issue date, recipient, part details, and quantity, with signature areas for the physical handover. The example shown is unsigned.

<p align="center">
  <a href="img/generate-voucher.png"><img src="img/generate-voucher.png" alt="Generated stock issue voucher with item details, handover acknowledgement, and blank signature areas" width="440"></a>
</p>

After signing, the PDF copy can be attached to the voucher and retrieved separately from the original.

</details>

The **Bons de sortie** register makes previous handovers searchable by voucher number, recipient, or person. Signature filtering and date sorting help staff locate records and follow up on paperwork.

<p align="center">
  <a href="img/voucher-section.png"><img src="img/voucher-section.png" alt="Voucher register with search, signature-status filtering, date sorting, and signed and unsigned voucher rows" width="720"></a>
</p>

*The register distinguishes filed signed copies from vouchers still awaiting them.*

## Application architecture

Django serves the interface and operational workflows. HTMX updates tables, filters, and dialogs; a batch pipeline imports OData exports, and document generation produces printable vouchers.

<p align="center">
  <a href="img/application-architecture.svg"><img src="img/application-architecture.svg" alt="Application architecture showing OData import, Django workflows, SQLite, file storage, and voucher PDF generation" width="800"></a>
</p>

*Logical application components; deployment details remain private.*

<details>
<summary><strong>Technology responsibilities and access model</strong></summary>

| Layer | Technologies | Responsibility |
| --- | --- | --- |
| Interface | Django templates, HTMX, Tailwind CSS | Pages, partial updates, and server-calculated SVG dashboard charts. |
| Application | Python, Django | Session authentication, forms, business rules, and persistence. Django was selected for its built-in security features. |
| Storage | SQLite, filesystem | Operational records, photos, and documents. SQLite keeps setup simple during early development. |
| Import preparation | pandas, Parquet, openpyxl | Prepare OData exports as Parquet for faster loading of large datasets; produce an Excel inspection copy. |
| Documents | python-docx, headless LibreOffice | Populate the voucher template and convert it to PDF during confirmation. |
| File validation | Pillow, pypdf | Validate uploaded images and PDFs. |

Signed-in accounts share the operational workflows. Voucher issuance and signed-copy upload record the acting account; temporary photos are scoped to their owner. Operator and manager are business roles rather than distinct application permission levels.

</details>

## Data architecture

Dispatch lines are resolved into **individually numbered units**, allowing partial returns and later stock issues to remain traceable. The schema connects those units to their return details, photo evidence, and voucher lines.

<p align="center">
  <a href="img/data-architecture-navy.png"><img src="img/data-architecture-navy.png" alt="Warehouse Manager database schema showing tables, fields, keys, cardinalities, and unit-to-voucher relationships" width="800"></a>
</p>

*Select the diagram to inspect the relationships at full size.*

<details>
<summary><strong>Identity, integrity, and recorded history</strong></summary>

- **Identity:** dispatch, part reference, and unit number identify a unit. Each voucher has a unique number; a unit can have at most one issue line.
- **Lifecycle:** awaiting return → returned and available → issued. Condition is recorded separately from availability.
- **Import integrity:** ambiguous service-order mappings and quantity reductions that would leave excess numbered units require reconciliation.
- **History:** return edits retain the first return date and append edit timestamps while replacing current values. Issue lines preserve a snapshot of unit details. Vouchers record the issuer and current signed copy’s uploader and upload time.
- **Storage:** operational records, temporary photo data, and the generated voucher document are stored in the database. Saved photos, archived PDFs, and signed copies use filesystem storage.

A service order can span several dispatches; each dispatch maps to at most one service order. History covers one return-to-issue lifecycle per unit, the latest location, and the current signed attachment.

</details>

## Engineering decisions

<img src="img/badges/tests.svg" alt="35 automated tests passing" height="28">

**Preserve identity across imports.** Reloading dispatch data reuses existing numbered units so their return records remain attached to the same unit.

**Keep stock and paperwork aligned.** Availability checks, unique submission identifiers, and database transactions coordinate stock issues with voucher generation.

<details>
<summary><strong>Implementation choices and tradeoffs</strong></summary>

**Data preparation.** OData exports are prepared as Parquet, chosen for faster loading of large datasets.

**Database choice.** SQLite keeps configuration and early development simple. PostgreSQL is a possible replacement if database performance becomes a bottleneck.

**Imports.** Dispatch and part consolidation determines unit quantities. Ambiguous mappings or incompatible reductions stop the import for reconciliation rather than silently changing tracked identities.

**Voucher issuance.** Availability is rechecked at confirmation; PDF generation and archiving occur during the operation. Tests cover duplicate submissions and rollback. Concurrent confirmations have not been verified in the evidence review.

**Photo evidence.** Photos are staged by user and unit and validated before save. Existing-photo deletion waits until a confirmed update commits. This requires draft expiry and cleanup logic.

**Signed documents.** Original and current signed PDFs remain separately accessible. Validation and stale-request checks protect updates; signature presence relies on the uploader’s confirmation. Complete signed-copy version history is outside the current implementation.

</details>

## Testing and limitations

The **35 passing automated tests** cover imports and reloads, dashboard calculations and filters, dispatch browsing, return updates and photos, voucher issuance and rollback, signed copies, and authentication.

The application supports return follow-up, recovered-unit lookup, documented stock issues, and retrieval of signed paperwork. Its current boundaries are:

- Stock figures cover recovered units; external tracking follows recorded events and manual follow-up.
- Each unit’s recorded lifecycle ends at stock issue, with partial history rather than a complete event log.
- Imports run in batches; scheduled refresh and additional reconciliation work remain unfinished.

Operational benefits have not been measured; no reduction in losses, fraud, or processing time is claimed.

---

*This repository showcases the project. The codebase is not shared because it contains sensitive business data and related business information. Identifying details in screenshots have been redacted.*
