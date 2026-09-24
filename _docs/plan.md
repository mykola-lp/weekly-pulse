# Weekly Project Feedback Tool: MVP Scope

## 1. Purpose
Give project owners a lightweight, recurring way to collect structured feedback from clients and stakeholders, and to see how sentiment trends week over week.

## 2. Users and roles
Multi-organization from day one. Each org's data is isolated.

| Role | Can do |
|---|---|
| **Admin** | Manage the org, members and roles; see all projects |
| **Project owner** | Create and manage own projects, stakeholders, schedules; view project dashboards |
| **View-only** | View dashboards for projects they are given access to |
| **Stakeholder** (external) | Submit feedback via email link; no account or login |

## 3. Core flow
1. A project owner creates a project and sets a **send day and time**.
2. The owner adds stakeholders by **manually entering email addresses**.
3. Each week, at the chosen time, every stakeholder receives an email with a unique link.
4. The stakeholder opens the link (no login) and rates three criteria, with an optional comment on each.
5. Non-responders get one **automatic reminder**; owners can also send a **manual nudge**.
6. Results appear on the project dashboard.

## 4. Survey
- **Fixed criteria (same for all projects):** Progress, Quality, Communication
- **Rating scale:** Traffic light (Red / Amber / Green)
- **Comments:** Optional free text per criterion

## 5. Privacy and anonymity
- Responses are **anonymous**. Owners see aggregate results only.
- The system stores **participation** (who responded) separately from **answers**, so reminders work without revealing who said what.
- Links use unique, unguessable tokens and are single-use per week.
- Consideration: with very few stakeholders, aggregates can de-anonymize individuals. Decide a minimum-response threshold before showing results (see Open Questions).

## 6. Dashboard
- Per-project view of weekly trends for each criterion (share of Red/Amber/Green per week)
- Response rate per week
- Comments shown without attribution

## 7. Scheduling and reminders
- Send day/time is configured per project by the owner.
- One automatic reminder to non-responders (timing to be defined, e.g. 48 hours later).
- Manual nudge button available to owners at any time before the survey closes.

## 8. Technical approach
- Web app: React frontend, backend API, relational database
- Transactional email provider for sends and reminders
- Background job scheduler for weekly sends and reminders
- Org-scoped data model with role-based access control

## 9. Open questions
1. What is the minimum number of responses before results are shown (anonymity threshold)?
2. When does a weekly survey link expire?
3. Reminder timing: how long after the initial email?
4. Should stakeholders be able to update a submitted response?
5. Email provider and sender-domain setup for each org?

## 10. Proposed non-goals for v1 (to confirm)
- Custom criteria or rating scales
- Slack/Teams delivery
- CSV import or self-join invite links for stakeholders
- Named or opt-in attributed responses
- Alerts on low ratings
