# TechSolve IT — Support Performance Dashboard

An end-to-end Power BI project analysing support ticket performance for TechSolve IT, covering data preparation, dashboard reporting, and a natural-language AI agent for querying the dataset.

## 📊 Project Overview

TechSolve IT's operations manager needed better visibility into support ticket performance: what issues are being raised, how they're being handled, and where there's room to improve. This project delivers:

1. **Cleaned, consolidated ticket data** combined with an external NZ Public Holidays dataset
2. **A 6-page Power BI dashboard** covering overview KPIs, resolution time, category breakdowns, ticket and customer insights, account value analysis, and team status
3. **An AI agent** (VS Code, GitHub Copilot, and a local Power BI MCP Server) that answers natural language questions directly against the live semantic model

## 🗂️ Part 1: Data Source, Combine & Prepare

**Primary dataset:** the ticket export provided for this exercise (ticket_id, customer/account details, category, priority, status, timestamps, resolution and SLA fields, CSAT, channel, technical metadata).

**External data source:** NZ Public Holidays, joined on ticket creation date to test whether holiday timing affects resolution speed.

**Key cleaning steps:**

- Replaced blanks with meaningful defaults, for example `company_name` became "Residential Customer" (confirmed by cross-checking against the `account_type` column), and `csat_score` was left as Null rather than 0, since 0 would misrepresent "no rating given" as "worst rating given"
- Standardised and consolidated the `category` field (TRIM, Clean and Capitalise, then merged near duplicate labels like "Bug" and "Bug Report" into 10 clean categories) while preserving the raw sub-category values for transparency
- Removed PII (`customer_email`, `billing_contact_email`); retained free-text fields (`issue_description`, `resolution_notes`) for the AI agent to use
- Corrected 9 tickets with invalid dates (years 1970 and 2099)
- Nulled resolution dates and times for tickets not yet Closed or Resolved, to avoid skewing resolution-time averages
- Rebuilt the SLA breach logic from scratch (`resolution_time_hours` vs `sla_target_hours`) after finding the supplied `sla_breached` field gave inconsistent results

## 📈 Part 2: Dashboard Pages

| Page | Contents |
| --- | --- |
| **Overview** | KPI cards (Total Tickets, Avg Resolution Hours, SLA Breach Rate, Open Tickets, Avg First Response), category/status breakdown, SLA breach by priority |
| **Performance Analysis** | Resolution time by category and priority, Team Performance table, public holiday impact |
| **Category & Sub-Category Breakdown** | Matrix drill-down of consolidated categories vs. raw sub-category labels, plus a data-quality callout card |
| **Ticket Insights** | Top 10 customers by ticket volume, tickets by region, tickets by industry, tickets by channel, tickets by account manager, tickets by service area |
| **Account Value vs. Service Performance** | Contract value vs. response, resolution and SLA metrics by top accounts, segment, and subscription tier |
| **Team Status** | Matrix of ticket status (Closed, In Progress, Open, Pending Customer, Resolved) by team and individual agent, plus Avg First Response and Avg Resolution Hour by category, broken out per team |

### Headline finding

**92% of resolved tickets breach their SLA target**, and this holds consistent across every dimension tested: category, team, priority, region, channel, service area, customer segment, subscription tier, and public holiday timing. No single factor explains it. This points to a **systemic capacity or process issue** (for example understaffing relative to volume, or no triage/escalation automation) rather than a localised one, reinforced by an average first-response time of 36.3 hours against a 4.16-hour target for Urgent tickets.

One real, non-flat finding: ticket volume is heavily skewed by team. Support handles 58,303 tickets, more than three times the next largest team (Billing, at 17,913). Despite this, SLA breach and escalation rates stay flat across teams (around 50%), suggesting Support may be understaffed relative to its workload even though the breach rate itself doesn't visibly spike there. This is worth investigating as a staffing or capacity question rather than a performance one.

## 🤖 Part 3: AI Agent

**Stack:** VS Code, GitHub Copilot (Agent mode), and the Power BI Modeling MCP Server (local), connected directly to the live semantic model behind the dashboard.

> Originally planned as a Microsoft Copilot Studio agent connected to a published Power BI dataset. I don't have access to the required Premium/Fabric workspace, so I used the local MCP connection instead.

**How it works:** the agent queries the actual DAX measures behind the dashboard, so answers are grounded in the same validated logic rather than being re-derived or guessed.

**Testing approach:** every agent response was cross-checked against Power BI Desktop and the raw Excel export before being trusted. Results were mixed:

| Metric | Verified value | Agent's answer | Match |
| --- | --- | --- | --- |
| Avg Resolution Hour | 120.72 | 120.72 | ✅ |
| SLA Breach Rate (after supplying business logic) | 92% | 92% | ✅ |
| Total Tickets | 100,851 | 100 | ❌ |
| Avg CSAT Score | 3.0 | 4.2 | ❌ |
| Avg First Response Time | 36.30 | 12.5 | ❌ |

The agent also produced a fluent but unsupported causal claim, stating Urgent tickets breached SLA at 98.57% and that breaches were "concentrated" in specific teams, which contradicted the dashboard's validated, flat breach-rate pattern across both priority and team (roughly 85 to 93% throughout, with no meaningful concentration in any one group).

**Takeaway:** AI agents connected to real data can still produce fluent, plausible-looking answers that are wrong or ungrounded. Every output was treated as a claim to verify, not a fact to report. See the full write-up in `TechSolve_Written_Review.docx` for the complete testing log and reasoning.

🎥 A screen recording (`screen_recording.mp4`) is included, showing a live walkthrough of the dashboard and a real-time Copilot Chat Q&A session against the connected semantic model, including examples of both correct and incorrect agent responses.

## 🛠️ Tools Used

- **Power BI Desktop**, for data modelling, DAX, and the dashboard build
- **Power Query**, for cleaning and transformation
- **VS Code with GitHub Copilot (Agent mode)**, as the Part 3 AI agent interface
- **Power BI Modeling MCP Server (local)**, to connect the agent to the live semantic model

## 📄 Full Written Review

See [`TechSolve_Written_Review.docx`](./TechSolve_Written_Review.docx) for the complete write-up: dataset sourcing rationale, process walkthrough, tool choices, challenges, time breakdown, AI tool usage, and assumptions made throughout the project.

## ▶️ How to Open

1. Clone or download this repository
2. Open `TechSolve_Dashboard.pbix` in Power BI Desktop (no Premium or Fabric workspace needed to view it locally)
3. Screenshots of each dashboard page and the AI agent test session are in `/screenshots`, for quick reference without needing to open the file
