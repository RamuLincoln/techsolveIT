# TechSolve IT — Support Performance Dashboard

An end-to-end Power BI project analysing support ticket performance for TechSolve IT, covering data preparation, dashboard reporting, and a natural-language AI agent for querying the dataset.

## 📊 Project Overview

TechSolve IT's operations manager needed better visibility into support ticket performance: what issues are being raised, how they're handled, and where there's room to improve. This project delivers:

1. **Cleaned, consolidated ticket data** combined with an external NZ Public Holidays dataset
2. **A 6-page Power BI dashboard** covering overview KPIs, resolution time, category breakdowns, team status, tickets and service performance analysis
3. **An AI agent** (VS Code + GitHub Copilot + local Power BI MCP Server) that answers natural language questions directly against the live semantic model

## 🗂️ Part 1 — Data Source, Combine & Prepare

**Primary dataset:** Ticket export (ticket_id, customer/account details, category, priority, status, timestamps, resolution/SLA fields, CSAT, channel, technical metadata)

**External data source:** NZ Public Holidays — joined on ticket creation date to test whether holiday timing affects resolution speed.

**Key cleaning steps:**

- Replaced blanks with meaningful defaults (e.g. `company_name` → "Residential Customer", `csat_score` left as Null rather than 0)
- Standardised and consolidated the `category` field (TRIM/Clean/Capitalize, then merged near-duplicate labels like "Bug"/"Bug Report" into 10 clean categories) while preserving raw sub-category values for transparency
- Removed PII (`customer_email`, `billing_contact_email`); retained free-text fields (`issue_description`, `resolution_notes`) for AI agent use
- Corrected 9 tickets with invalid dates (years 1970/2099)
- Nulled resolution dates/times for tickets not yet Closed or Resolved, to avoid skewing resolution-time averages
- Rebuilt SLA breach logic from scratch (`resolution_time_hours` vs `sla_target_hours`) after finding the supplied `sla_breached` field gave inconsistent results

## 📈 Part 2 — Dashboard Pages

| Page                                      | Contents                                                                                                                                              |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Overview**                              | KPI cards (Total Tickets, Avg Resolution Hours, SLA Breach Rate, Open Tickets, Avg First Response), category/status breakdown, SLA breach by priority |
| **Performance Analysis**                  | Resolution time by category & priority, Team Performance table, public holiday impact                                                                 |
| **Category & Sub-Category Breakdown**     | Matrix drill-down of consolidated categories vs. raw sub-category labels, plus a data-quality callout card                                            |
| **Customer & Regional Insights**          | Top 10 customers, tickets by region, tickets by channel, tickets by service area                                                                      |
| **Account Value vs. Service Performance** | Contract value vs. response/resolution/SLA metrics by top accounts, segment, and subscription tier                                                    |
| **SLA Breach — Key Influencers**          | Built-in Power BI Key Influencers + Smart Narrative, used as an independent check on manual findings                                                  |

### Headline finding

**92% of resolved tickets breach their SLA target**, and this is consistent across every dimension tested — category, team, priority, region, channel, service area, customer segment, subscription tier, and public holiday timing. No single factor explains it; This points to a **systemic capacity or process issue** (e.g. understaffing relative to volume, no triage/escalation automation) rather than a localised one — reinforced by an average first-response time of 36.3 hours against a 4.16-hour target for Urgent tickets.

## 🤖 Part 3 — AI Agent

**Stack:** VS Code + GitHub Copilot (Agent mode) + Power BI Modeling MCP Server (local), connected directly to the live semantic model behind the dashboard.

> Originally planned as a Microsoft Copilot Studio agent connected to a published Power BI dataset. I dont have access to the required Premium/Fabric workspace, so decided to use the local MCP connection.

**How it works:** the agent queries the actual DAX measures behind the dashboard, so answers are grounded in the same validated logic — not re-derived or guessed.

**Testing approach:** every agent response was cross-checked against Power BI Desktop and the raw Excel export before being trusted. Results were mixed:

| Metric                                           | Verified value | Agent's answer | Match |
| ------------------------------------------------ | -------------- | -------------- | ----- |
| Avg Resolution Hour                              | 120.72         | 120.72         | ✅    |
| SLA Breach Rate (after supplying business logic) | 92%            | 92%            | ✅    |

The agent also produced a fluent but unsupported causal claim (Urgent tickets breaching at 98.57%, breaches "concentrated" in specific teams) that contradicted the dashboard's validated, flat breach-rate pattern.

🎥 A screen recording (screen_recording.mp4) is included showing a live walkthrough of the dashboard and a real-time Copilot Chat Q&A session against the connected semantic model, including examples of both correct and incorrect agent responses.

## 🛠️ Tools Used

- **Power BI Desktop** — data modelling, DAX, dashboard build
- **Power Query** — cleaning and transformation
- **VS Code + GitHub Copilot (Agent mode)** — Part 3 AI agent interface
- **Power BI Modeling MCP Server (local)** — connects the agent to the live semantic model

## 📄 Full Written Review

See TechSolve_Written_Review.docx for the complete write-up: dataset sourcing rationale, process walkthrough, tool choices, challenges, time breakdown, AI tool usage, and assumptions made throughout the project.

## ▶️ How to Open

1. Clone/download this repository
2. Open `TechSolve_Dashboard.pbix` in Power BI Desktop (Power BI Desktop required — no Premium/Fabric workspace needed to view locally)
3. Screenshots of each dashboard page and AI agent test session are in `/screenshots` for quick reference without needing to open the file
