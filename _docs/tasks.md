# Task Backlog: Weekly Project Feedback Tool (v1)

Stack: Next.js (App Router, TypeScript), Postgres, Prisma, Clerk (auth and orgs), Resend (email), Inngest (scheduling), Recharts (charts), Vitest (tests).

Product summary: project owners add stakeholders to a project; each week stakeholders get an email with a no-login link to rate Progress, Quality and Communication (red/amber/green) with optional comments. Responses are anonymous; owners see aggregates only on a dashboard.

## 1. Set up an empty project with a passing test
Goal: Have a runnable Next.js TypeScript project with a green test suite.
Description: Create a new Next.js app (App Router, TypeScript, ESLint) and add Vitest with one trivial passing test. Add npm scripts for `dev`, `build`, `lint` and `test`, and a README with setup instructions. No features yet.

## 2. Add a CI pipeline
Goal: Run lint, type check and tests automatically on every pull request.
Description: Add a GitHub Actions workflow that installs dependencies, then runs lint, `tsc --noEmit` and the test suite on pushes and pull requests. Cache dependencies for speed. Assumes an existing Next.js project with `lint` and `test` npm scripts.

## 3. Connect Postgres and Prisma
Goal: Have a working database connection with migrations in place.
Description: Add Prisma to the Next.js project, configure `DATABASE_URL` through environment variables, and add a `.env.example`. Provide a local Postgres setup (e.g. docker-compose) and a migration workflow, and verify the connection with a small test or script.

## 4. Define the core schema: orgs, members, projects
Goal: Model organizations, memberships with roles, and projects in the database.
Description: Add Prisma models for Organization, Membership (roles: ADMIN, PROJECT_OWNER, VIEW_ONLY) and Project (name, owner, send day, send time, timezone). Every project must belong to exactly one organization. Include a migration and a seed script with sample data.

## 5. Integrate Clerk authentication
Goal: Let users sign up, sign in and belong to an organization.
Description: Add Clerk to the Next.js app with sign-in and sign-up pages and middleware protecting all routes except public survey routes under `/s/*`. On first sign-in, sync the Clerk user and organization into the local Organization and Membership tables. Document the required environment variables.

## 6. Build a role-based access helper
Goal: Provide one reusable way to check what the current user may do.
Description: Create server-side helpers that resolve the current user's organization and role and expose checks such as `canManageOrg`, `canManageProject` and `canViewProject`. Admins can do everything in their org, project owners manage their own projects, and view-only users can only read. Cover the rules with unit tests.

## 7. Build project create and edit screens
Goal: Let admins and owners create and edit projects.
Description: Add pages to list, create, edit and delete projects within the user's organization. The form covers name, send day of week, send time and timezone. Enforce role permissions server-side so view-only users cannot change anything.

## 8. Manage stakeholders per project
Goal: Let owners add and remove stakeholder emails on a project.
Description: Add a Stakeholder model (project, email, status) and a project sub-page to add, list and remove stakeholders by email address with validation and duplicate protection. Emails are entered manually, one at a time. Only users allowed to manage the project can make changes.

## 9. Design the survey data model with separated participation
Goal: Store who responded separately from what they answered.
Description: Add models for SurveyRound (project, week, open/closed), Participation (round, stakeholder, responded flag, token) and Response (round, three ratings, optional comments). Response must have no foreign key to Stakeholder or Participation, so answers cannot be traced back to a person. Include a migration and a short note documenting this design decision.

## 10. Generate and validate survey link tokens
Goal: Give each stakeholder a unique, unguessable, single-use link per round.
Description: Implement functions that create a cryptographically random token per participation and validate it, returning the round and participation or a clear error (invalid, expired, already used). Store only a hash of the token in the database. Cover all outcomes with unit tests.

## 11. Build the public survey page
Goal: Let a stakeholder open a link and see the survey without logging in.
Description: Create a route `/s/[token]` that validates the token and shows the project name and three criteria (Progress, Quality, Communication), each with a red/amber/green selector and an optional comment box. Show friendly pages for invalid, expired and already-submitted links. The page must be mobile friendly.

## 12. Implement survey submission
Goal: Save a submitted survey anonymously and mark the stakeholder as responded.
Description: Add a server action or API route that validates the token, requires a rating for each criterion, writes a Response row with no link to the person, and in the same transaction marks the participation as responded and the token as used. Reject double submissions. Cover it with integration tests.

## 13. Build the email sending module
Goal: Send transactional emails through Resend.
Description: Create a small email service wrapping Resend with a typed `sendEmail` function and an environment-based sender address. Add templates for the weekly survey request and the reminder, each containing the project name and the survey link. Make the provider mockable so tests do not send real emails.

## 14. Schedule the weekly survey send
Goal: Send each project's survey on the day and time its owner chose.
Description: Use Inngest to run a recurring job that finds projects due for sending in their own timezone, creates a SurveyRound, creates a Participation with a token for each stakeholder, and sends the survey email. Make the job idempotent so a retry never creates a duplicate round or sends duplicate emails.

## 15. Send automatic reminders
Goal: Remind stakeholders who have not responded.
Description: Add an Inngest job that, a set time after a round starts (default 48 hours, configurable by a constant), emails a reminder to participations not yet marked as responded. Send at most one automatic reminder per stakeholder per round. Skip closed rounds.

## 16. Add a manual nudge for owners
Goal: Let an owner remind non-responders on demand.
Description: Add a "Send nudge" button on the project page for the current open round that emails only stakeholders who have not responded. Show the number of people who will be nudged before sending. Limit nudges to a sensible cap per round, and restrict the action to users who can manage the project.

## 17. Close survey rounds and handle link expiry
Goal: Stop accepting responses after a round ends.
Description: Add a job that closes each round after a defined window (default 7 days, or when the next round starts). Closed rounds make survey links show an "expired" page and reject submissions. Include tests for the boundary times.

## 18. Build the dashboard trend chart
Goal: Show weekly rating trends per criterion for a project.
Description: Create a project dashboard page with a Recharts chart showing, per week, the share of red, amber and green ratings for each of the three criteria. Data comes from aggregate queries over Response and never exposes individual rows. Only users who can view the project can open it.

## 19. Add response rate and anonymous comments to the dashboard
Goal: Show how many stakeholders responded and what they wrote.
Description: On the project dashboard, show the response rate per week (responded participations over total) and a list of comments without any attribution, most recent first. Comments are read from Response rows only. Show an empty state when there is no data yet.

## 20. Enforce a minimum-response anonymity threshold
Goal: Avoid revealing individual answers when very few people respond.
Description: Add a configurable minimum number of responses (default 3) below which the dashboard hides ratings and comments for that round and shows a message instead. Apply the rule in the data-access layer so no screen can bypass it. Cover the threshold behavior with unit tests.

## 21. Build member and role management for admins
Goal: Let org admins invite members and assign roles.
Description: Add an admin-only page listing organization members with their role (admin, project owner, view-only), allowing role changes and member removal. Use Clerk organization invitations for onboarding new members. Prevent removing or demoting the last admin.

## 22. Give view-only users project access
Goal: Control which projects a view-only member can see.
Description: Add a way to grant view-only members access to specific projects, and filter project lists and dashboards accordingly. Admins can see all projects in the organization, and owners see the projects they own. Test each role's visibility.

## 23. Prepare production deployment
Goal: Deploy the app with all services configured.
Description: Configure hosting (e.g. Vercel), a managed Postgres database, and environment variables for Clerk, Resend and Inngest. Run migrations as part of the release, set up the sender domain for email, and document the deployment steps and required secrets. Verify with a smoke test on the deployed URL.
