S42010/not.md

# SAP S/4HANA — Production Planning (PP)
**Course S42010** | Exploring Business Processes in SAP S/4HANA Production Planning

---

## Contents
1. Lead Time & Delivery Time · Push & Pull
2. SAP Integrated Business Planning (IBP)
3. Demand Management & PIR
4. Requirements Types
5. Planning Strategies — Make-to-Stock (10, 40, 70)
6. Consumption Modes
7. Planning Strategies — Make-to-Order (20, 50, 60, 82)
8. MRP-Live & Interactive MRP
9. Planning Run
10. Low-Level Code & Net Change Planning
11. Firming Period & Planning Time Fence
12. Evaluating Results & MRP Cockpit
13. Interview Q&A
14. Glossary

---

# 1. Lead Time & Delivery Time · Push & Pull

**Lead Time** = time from order placement to production completion (raw material + manufacturing).  
**Delivery Time** = time from production completion to customer receipt.

```
Order Placed → [Lead Time] → Product Ready → [Delivery Time] → Customer Receives
```

---

**Push Strategy** — production starts based on forecast, before any real order arrives. Stock is built in advance. Fast delivery, but excess inventory risk.  
SAP: MRP / PIR-based planning, Strategies 10 & 40.

**Pull Strategy** — production only starts when an actual customer order is received. Low inventory, but longer lead time.  
SAP: Kanban, JIT, Strategy 20.

| Criteria | Push | Pull |
|---|---|---|
| Trigger | Forecast | Actual order |
| Inventory | High | Low |
| Delivery speed | Very fast | Slower |
| Risk | Overstock | Late delivery |

> 💡 Most companies use a **hybrid**: raw materials pushed into stock, final assembly pulled by actual orders. The boundary between the two is the **Decoupling Point**.

---

# 2. SAP Integrated Business Planning (IBP)

SAP IBP is a cloud-based platform that brings demand forecasting, supply planning, and supply chain optimization into a single tool.

| Module | Purpose |
|---|---|
| IBP for Demand | AI-driven demand forecasting (seasonality, promotions) |
| IBP for Supply | Supply planning under capacity and supplier constraints |
| IBP for Inventory | Calculates optimal stock levels |
| IBP for S&OP | Aligns sales, finance, and operations in one plan |
| IBP for Response | Real-time reaction to sudden changes |

---

# 3. Demand Management & PIR

**Demand Management** combines two demand types — forecasts and actual orders — into a **Demand Program**, which feeds directly into MRP.

| Step | What happens |
|---|---|
| 1 | Forecast → PIR created |
| 2 | Real customer orders entered |
| 3 | PIR + orders = Demand Program |
| 4 | Demand Program → MRP input |
| 5 | MRP calculates what to order and when |

---

**Planned Independent Requirement (PIR)** — a forecast-based requirement with no link to any actual customer order. The company creates it independently based on expected demand.

> ⚠️ PIR says *"this much is needed"*. MRP decides *how to cover it*.

| | PIR | Sales Order |
|---|---|---|
| Created by | Company (forecast) | Customer (real order) |
| Certainty | Estimate | Confirmed |
| Reduces when | Goods Issue (S10) / Sales Order (S40) | Goods Issue |

---

# 4. Requirements Types

Three types of requirements make up the Demand Program:

**PIR** — forecast-based stock requirement. No actual order needed.  
> *Example: Historical data + trade fair insight → 200 bicycles expected → PIR entered → MRP plans production.*

**Sales Order** — real customer order, may include special requirements (e.g. custom logo).

**Stock Transfer Requirement** — internal demand from another plant within the same company network (cross-plant).  
> *Example: Istanbul DC requests 50 bicycles from Ankara plant. Plant PIR 100 + transfer 50 = 150 unit Demand Program.*

---

# 5. Planning Strategies — Make-to-Stock

The planning strategy defines when production starts and how sales orders interact with MRP.

| Strategy | Production Start | Stock Level | Type |
|---|---|---|---|
| Make-to-Stock | Based on forecast | High — finished goods | Push |
| Final Assembly (S40) | Forecast + adjusts to orders | Medium | Hybrid |
| Make-to-Order | Only on real order | Very low | Pull |

---

## Strategy 10 — Pure Make-to-Stock
Production is 100% driven by PIR. Sales orders are **invisible to MRP**. PIR is only reduced at Goods Issue (physical shipment).

| Step | Event | PIR |
|---|---|---|
| 1 | PIR: produce 50 bicycles | 50 |
| 2 | MRP runs → 50 produced | 50 |
| 3 | Sales Order: 30 received | 50 — MRP ignores it |
| 4 | 30 shipped from warehouse | 50 — not yet reduced |
| 5 | Goods Issue posted | 50 → 20 ✅ |

```
PIR Remaining = PIR Initial − Goods Issue Quantity
Net Requirement = PIR − Current Stock
```

---

## Strategy 40 — Planning with Final Assembly
Same as S10 but sales orders **consume PIR immediately**. If orders exceed PIR, MRP automatically plans the gap.

| Scenario | PIR | Sales Order | PIR After | MRP |
|---|---|---|---|---|
| Normal | 50 | 30 | 20 | Continues for 20 |
| Excess | 50 | 55 | 0 (gap: 5) | Auto-plans 5 extra |
| No orders | 50 | 0 | 50 | Plans 50 |

```
PIR Remaining = PIR − Sales Order
Net Requirement = MAX(0, Sales Order − PIR)
```

---

## Strategy 70 — Planning at Assembly Level
No PIR for the finished product. PIR is created at **component/assembly level**. Final assembly only happens when a customer order specifies the exact variant. Best for highly configurable products.

```
Assembly PIR = Finished Product Forecast × BOM Quantity
Assembly PIR Remaining = Assembly PIR − (Sales Order × BOM Quantity)
```

> *Example — 50 bicycle forecast:*  
> *Wheel (×2): 100 units PIR | Frame (×1): 50 units PIR*  
> *10 bicycle order arrives → Wheel: 100 − 20 = 80 | Frame: 50 − 10 = 40*

| | S10 | S40 | S70 |
|---|---|---|---|
| PIR level | Finished product | Finished product | Assembly |
| Sales order affects MRP | No | Yes | Yes |
| PIR reduces at | Goods Issue | Order arrival | Assembly level |
| Stock held | Finished goods | Finished goods | Components only |
| Push/Pull | Pure Push | Hybrid | Near Pull |

---

# 6. Consumption Modes

In Strategies 40 and 70, when a sales order arrives, the system must decide **which period's PIR to consume**. Consumption Mode sets the direction.

| Mode | Code | Direction | Use case |
|---|---|---|---|
| Backward | 1 | ← Previous periods | Late arriving order |
| Forward | 3 | → Future periods | Early arriving order |
| Backward + Forward | 2/4 | ← then → | Most common, general purpose |
| Periodic-Specific | 5 | Same period only | Strict period boundaries required |

---

# 7. Planning Strategies — Make-to-Order

| Strategy | Code | PIR | Description |
|---|---|---|---|
| Make-to-order | 20 | None | Pure Pull — no production without an order |
| Without final assembly | 50 | None (assembly) | Components start immediately; final assembly on order |
| With planning material | 60 | Yes (planning material) | High-level variant management |
| Assembly processing | 82 | None | Order-based assembly |

---

## Strategy 20 — Make-to-Order
No PIR. Production starts only on a real sales order. Each order is planned and produced independently.

Three rules:
- Orders are never consolidated by MRP — each is separate.
- Produced goods go into **customer-specific stock** — cannot be used for another order.
- **Lot-for-lot**: produce exactly the ordered quantity.

---

## Strategy 50 — Planning without Final Assembly
No finished product PIR. But when a sales order arrives, SAP **immediately explodes the BOM** and starts component procurement — without waiting for the final assembly trigger.

> The sales order acts as the PIR for components: *"don't assemble yet, but start all parts now."*  
> Delivery time is shorter than S20 because components are ready when assembly begins.

| | Strategy 20 | Strategy 50 |
|---|---|---|
| Component trigger | After order (delayed) | At order (immediate BOM) |
| Delivery speed | Slow | Faster |
| Use case | Fully custom | Partially predictable |

---

# 8. MRP-Live & Interactive MRP

**MRP goal:** right material, right source, right time.  
**MRP-Live** is the real-time successor to classic MRP, running on the SAP HANA in-memory database.

## MRP Run — 4 Steps

| Step | Process | What happens |
|---|---|---|
| 1 | READ | Collects stock, open orders, PIRs, BOMs, lead times |
| 2 | NETTING & LOT SIZING | Net Req. = Gross Req. − Stock; applies lot sizing rules |
| 3 | BOM EXPLOSION | Calculates all components per product using low-level codes |
| 4 | WRITE | Creates Planned Orders, Purchase Requisitions, Schedule Lines |

## Classic MRP vs MRP-Live

| | Classic MRP | MRP-Live |
|---|---|---|
| Frequency | Weekly batch | Near real-time |
| Speed | Hours | 10× faster |
| Data | Snapshot (goes stale) | Real-time |
| Platform | On-premise ERP | SAP S/4HANA (HANA DB) |

## Key Fiori Applications

| App | Purpose |
|---|---|
| MRP Cockpit | Full organization MRP status; KPI tiles |
| Monitor Material Coverage | Lists shortages; quick-create Purchase Req. or Planned Order |
| Stock/Requirements List | Full material timeline; real-time editable |
| Manage Material Coverage | Proactive coverage actions |

| Classic Transaction | Purpose |
|---|---|
| MD04 | Stock/Requirements List — single material |
| MD05 | MRP List — single material |
| MD06 / MD07 | Bulk MRP / S&R List access |
| MD09 | Pegging — traces which supply covers which demand |

---

# 9. Planning Run

| Type | Scope | When |
|---|---|---|
| Total Planning | Entire plant + full BOM explosion | Weekly / overnight |
| Single-Item Single-Level | 1 material, 1 BOM level | Emergency |
| Single-Item Multi-Level | 1 material, all BOM levels | Detailed analysis |
| Planning Scope | Multiple plants | Cross-plant dependencies |

**Online** — planner waits; instant result. For small plants or urgent cases.  
**Background Job** — scheduled overnight. Requires Variant + Schedule setup.

**Parallel Processing** — splits planning across multiple processors. Faster for large plants. If one material errors, others continue unaffected (robustness).

## Control Parameters

| Parameter | Options |
|---|---|
| Regenerative Planning | Replans everything from scratch (weekly) |
| Net Change (NETCH) | Only changed materials (daily) |
| Scheduling | 1 = basic dates / 2 = lead time + capacity |
| Planning Mode | 1 = update existing / 3 = delete and recreate |

---

# 10. Low-Level Code & Net Change Planning

## Low-Level Code
Every material receives the code of the **lowest level it appears at** across all BOMs. MRP plans top-down, following these codes.

| Code | Example | Planned |
|---|---|---|
| 000 | Finished products | First |
| 001 | Sub-assemblies | Second |
| 002 | Raw components | Last |

> Why top-down? You can't know how many components are needed until you know how many finished products are planned. Sequence is always: Finished Product → Assembly → Component.

## Net Change Planning (NETCH)
Only plans materials with an MRP-relevant change since the last run. Changes are flagged in the **Planning File**.

| Change type | Auto-flagged? |
|---|---|
| Sales order created | Yes ✅ |
| Stock movement | Yes ✅ |
| Procurement type changed | Yes ✅ |
| Routing changed | No ⚠️ — manual |

After the planning run, Planning File entries and NETCH flags are automatically cleared.

---

# 11. Firming Period & Planning Time Fence

The Firming Period prevents MRP from automatically changing orders that production has already started on.

**Planning Time Fence** — set in days in Material Master. Within this window, Planned Orders are locked.  
**Manual Firming Date** — planner manually sets a firming date per order.

> If both are active, whichever expires later applies (longer protection wins).

## Firming Types

| Type | Existing orders | New shortages within fence |
|---|---|---|
| 0 | Not firmed | Created normally |
| 1 | Auto-firmed ✅ | Created, pushed to end of fence |
| 2 | Auto-firmed ✅ | Not created — shortage stays open |
| 3 | Not firmed (manual firming respected) | Created, pushed to end of fence |
| 4 | Not firmed (manual firming respected) | Not created — shortage stays open |

> When MRP detects a needed change inside the fence, it cannot act — it raises an **Exception Message** instead. The planner reviews and decides manually.

---

# 12. Evaluating Results & MRP Cockpit

A **shortage** occurs when total demand exceeds total supply for a material.

| Demand side | Supply side |
|---|---|
| Sales Orders | Current inventory |
| Stock Transfer Requirements | Production Orders |
| Dependent requirements | Purchase Orders |
| PIR (forecast) | Firmed Planned Orders |

## MRP Cockpit Screens

| Screen | Purpose |
|---|---|
| Uncovered Demand List | Orders that cannot be fulfilled |
| Material List | MRP summary across all materials |
| Late Supply List | Supplies arriving too late |
| MRP Elements (MD04-like) | Full Stock/Requirements detail per material |

## Key Features

- **Color coding:** Green = OK · Yellow = At risk · Red = Shortage
- **Simulation:** Test a proposed fix before applying it
- **Single Fiori page** replaces MD04, MD06, MD07 and multiple other transactions
- **SAP HANA benefit:** MRP runtime reduced by ~50%; real-time data throughout

---

# 13. Interview Q&A

**Q1: Lead Time vs Delivery Time?**  
Lead Time covers everything from order placement to production completion — raw material sourcing plus manufacturing. Delivery Time is only the transport leg from warehouse to customer. In SAP backward scheduling, Lead Time is the critical input: ignore it and materials arrive late, stopping production.

---

**Q2: Push vs Pull?**  
Push produces based on forecast before orders arrive — fast delivery but inventory risk. Pull produces only on a real order — low inventory but longer delivery. Most companies use a hybrid: push raw materials into stock, pull final assembly on order. The switchover point is the Decoupling Point.

---

**Q3: What is PIR and how does it relate to MRP?**  
A PIR is a forecast quantity entered by the company — not linked to any actual customer order. It tells MRP what to plan for. MRP reads the PIR, calculates net requirements, explodes the BOM, and generates planned orders or purchase requisitions.

---

**Q4: Differences between Strategies 10, 40, and 70?**  
S10 is pure make-to-stock: production is PIR-driven, sales orders are invisible to MRP, PIR only reduces at Goods Issue. S40 adds sales order visibility: orders consume PIRs immediately, and if orders exceed the PIR, MRP auto-plans the gap. S70 shifts planning to assembly level: PIRs are created for common components, not the finished product, so final assembly is only triggered when a customer specifies the variant — ideal for configurable products.

---

**Q5: Make-to-Stock vs Make-to-Order (Strategy 20)?**  
Make-to-Stock uses PIRs and holds finished goods in a general warehouse available to any customer. Strategy 20 has no PIRs — production only starts on a real order, output goes into customer-specific stock (no reallocation possible), and lot-for-lot sizing means exactly the ordered quantity is produced.

---

**Q6: Strategy 50 vs Strategy 20?**  
Both are make-to-order with no finished product PIR. The difference: in S50, when a sales order arrives, SAP immediately explodes the BOM and starts component procurement — without waiting. Final assembly still only happens on order, but components are ready earlier, shortening delivery time. Use S50 when components are predictable but the final variant is not.

---

**Q7: What are Consumption Modes?**  
In Strategies 40 and 70, an incoming sales order must consume a PIR from some period. The Consumption Mode determines which: Mode 1 goes backward (late orders), Mode 3 goes forward (early orders), Mode 2/4 combines both (most common), Mode 5 stays in the same period only (strict boundaries). Set per material in SAP Customizing.

---

**Q8: MRP-Live vs classic MRP?**  
Classic MRP ran as a weekly or nightly batch job — a snapshot that was already outdated by morning. MRP-Live runs on SAP HANA, is up to 10× faster, uses 5× less memory, and provides real-time visibility. Planners work in the MRP Cockpit: one Fiori screen to see shortages, simulate fixes, and apply changes — replacing the old multi-transaction workflow.

---

**Q9: Planning Time Fence and Firming Types?**  
The Planning Time Fence (set in days in Material Master) protects planned orders from automatic MRP changes — useful when production has already started. Firming Types define the behavior: Type 1 auto-firms existing orders and pushes new shortages to the fence boundary; Type 2 auto-firms but suppresses new orders inside the fence; Types 3 and 4 mirror these but only protect manually firmed orders. When MRP needs to change something inside the fence, it raises an Exception Message instead.

---

**Q10: Net Change Planning and the Planning File?**  
NETCH planning only processes materials flagged with an MRP-relevant change since the last run. When a change occurs — new sales order, stock movement, procurement type change — SAP writes a NETCH indicator to the Planning File. The next NETCH run only processes flagged materials, making it far faster than Regenerative Planning, which replans everything. Flags are cleared automatically after each run.

---

**Q11: Low-Level Code?**  
The Low-Level Code is the deepest level at which a material appears across all BOMs in the system. MRP always plans the lowest code number first (finished product = 000) and works downward. This guarantees that when component requirements are calculated, all higher-level demands are already known — no missed or double-counted requirements.

---

**Q12: How do you evaluate and resolve a shortage in S/4HANA?**  
Open the MRP Cockpit → Monitor Material Coverage to see all shortages color-coded by severity. Click the affected material → Manage Material Coverage for the Stock/Requirements List with exact shortage quantity and date. The system proposes fixes: new PO, reschedule existing supply, or pull stock from another location. Simulate each option to see the ripple effect before committing. Everything happens in one Fiori app — no switching between MD04, MD07, and separate transaction screens.

---

# 14. Glossary

| Term | Full Name | Meaning |
|---|---|---|
| PIR | Planned Independent Requirement | Forecast-based requirement; not linked to any actual order |
| MRP | Material Requirements Planning | Calculates what to produce/procure, when, and how much |
| MRP-Live | MRP-Live (S/4HANA) | Real-time MRP running on SAP HANA |
| IBP | Integrated Business Planning | SAP's cloud-based advanced planning platform |
| S&OP | Sales and Operations Planning | Aligns sales, finance, and operations |
| BOM | Bill of Materials | Full list of components and quantities for one unit of product |
| MPS | Master Production Schedule | High-level production plan |
| MTO | Make-to-Order | Production triggered by actual order only |
| MTS | Make-to-Stock | Forecast-driven production; goods held in stock |
| Goods Issue | Goods Issue | Recording physical departure of goods from warehouse in SAP |
| Lead Time | Lead Time | Order placement → production complete |
| Delivery Time | Delivery Time | Production complete → customer receipt |
| Decoupling Point | Decoupling Point | Where Push strategy ends and Pull strategy begins |
| Stock Transfer Req. | Stock Transfer Requirement | Internal demand from another plant in the same company |
| Availability Check | Availability Check | Automatic stock check at time of order |
| Backorder | Backorder | Unfulfilled order carried forward due to stock shortage |
| Firming Period | Firming Period | Protected window where MRP cannot auto-change orders |
| Planning Time Fence | Planning Time Fence | Firming period boundary defined in days in Material Master |
| NETCH | Net Change Planning | MRP mode that only replans materials with recent changes |
| Planning File | Planning File | SAP record of materials flagged for the next NETCH run |
| Low-Level Code | Low-Level Code | Lowest BOM level at which a material appears — sets MRP sequence |
| Customer Stock | Customer Stock | S20 stock reserved for one specific customer; cannot be reallocated |
| Lot-for-lot | Lot-for-lot | Produce exactly the ordered quantity — nothing more |
| Bullwhip Effect | Bullwhip Effect | Small demand changes amplifying into large supply chain swings |
| PP/DS | Production Planning / Detailed Scheduling | Capacity planning layer on top of MRP-Live |
