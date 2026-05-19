# POM Vocabulary

Authoritative term index. Skills, workflows, and templates use this vocabulary consistently — never substitute generic Agile/Scrum terms where a POM term exists.

| Term | Meaning |
|---|---|
| **WSJF** | Weighted Shortest Job First. `WSJF = Cost of Delay ÷ Job Size`. See [`wsjf-formula.md`](wsjf-formula.md). |
| **Cost of Delay** | Sum of User-Business Value + Time Criticality + Risk Reduction & Opportunity Enablement. Each dimension scored on modified Fibonacci (1, 2, 3, 5, 8, 13, 21). |
| **Job Size** | Effort to deliver end-to-end through a pod, including discovery and runway. Not story points. |
| **Use Case Backlog** | Portfolio-shared, WSJF-prioritized intake. Lives at `intake/use-case-backlog/` in a POM repo. |
| **Use Case (UC)** | A single stakeholder request, scored. File format `UC-NNNN-<slug>.md`. |
| **Disposition** | A formal decision on a UC: **kill**, **park**, **route**, or **split**. See [`disposition-rules.md`](disposition-rules.md). |
| **Discovery Backlog** | Per-product folder of opportunities being shaped by the Trio. Lives at `products/<x>/discovery-backlog/`. |
| **Discovery item (DISC)** | A single opportunity being shaped. File format `DISC-YYYY-MM-<slug>.md`. References its originating UC. |
| **Four-question gate** | The four questions that must be answered before a Discovery item promotes to the Product Backlog: **desirable**, **viable**, **feasible**, **ethical/compliant**. See [`four-question-gate.md`](four-question-gate.md). |
| **Product Backlog** | Per-product folder of shaped items ready for pod pull. Lives at `products/<x>/product-backlog/`. |
| **Product Backlog item (PB)** | A single shaped, ready-to-pull item. File format `PB-YYYY-MM-<slug>.md`. Defined as a **vertical slice**, not a horizontal task list. |
| **Product Trio** | The three roles that jointly own a product's backlog/roadmap/runway: **Product Manager (PM)**, **Product Designer (PD)**, **Technical Leader (TL)**. POs do **not** belong to the Trio. |
| **Product Enablement Team** | Portfolio-shared supporting cast: **Vendor Manager**, **Data Architect**. Supports every Trio. |
| **Pod** | Cross-functional team of **2–7 members** + Scrum Master (SM) + Product Owner (PO). Owned by exactly one product (default). See [`pod-composition-rules.md`](pod-composition-rules.md). |
| **PO Sync** | Bi-weekly ceremony where the Trio + all pod POs re-order the Product Backlog. Pod POs do not unilaterally re-prioritize between syncs. |
| **Architectural Runway** | Technical foundation laid down ahead of where pods are about to ship. ADRs + patterns + a living `runway-plan.md`. Owned by the TL, built by pods. |
| **Vertical slice** | A thin end-to-end customer-journey increment. The unit of value in this model — not stories, not horizontal layers. |
| **Platform Services (Layer 2)** | Portfolio-shared services consumed by products: provisioning, templates, governance, data quality. Consumed, not mandated. |
| **Enabling Functions (Layer 3)** | Portfolio-shared practices and standards: AI ethics, security, agility coaching, metrics. Make the right thing the easy thing. |
| **Release Planning** | Periodic cross-pod coordination event where pods align on dependencies and slice sequencing. |
| **Decision Log** | Append-only record of standards-level or process-level decisions (dispositions, AI ethics precedents, runway ADRs). Future work inherits these as precedents. |

## Word we deliberately avoid

| Avoid | Use instead |
|---|---|
| "Story" / "User story" | **Vertical slice** (in Product Backlog) or **Discovery item** (in shaping). |
| "Epic" | **Use Case** at intake; **Product Backlog item** after shaping. |
| "Sprint backlog" | **Pod backlog**. |
| "Team lead" | **Pod PO** or **Pod SM** depending on responsibility. |
| "Tech lead" of a product | **Technical Leader (TL)** — explicitly part of the Trio. |
| "PM owns the backlog" | **The Trio owns the backlog.** |
