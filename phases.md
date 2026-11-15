# Anjaam — development phases

Product name: **Anjaam** (انجام). The line under the name is "Bring every task to completion." The README keeps the portal title, Task Management Tool, so the submission matches the assignment.

This file is the build order for [docs/project-brief.md](docs/project-brief.md). It does not add behavior the brief left out. One task is one commit. No commit goes directly to `main`.

## Overview

Seven phases, 35 tasks. Phase 1 leaves a buildable empty solution. Phases 2 and 3 add the schema and the API host. From Phase 4 on, each feature's API is committed and tested before that feature's screens. The task list is the exception: it is designed once, after filter, export, and import exist.

A screen is never coded before its own design is approved. An API commit does not approve a screen. The logo is not designed in this file. PHASE-04-TASK-04 shows original concepts and waits for approval before any mark is added.

## Strategy

The solution is Domain, Application, Infrastructure, Api, unit tests, integration tests, and a React client in `client/`.

- Domain has no Entity Framework and no ASP.NET Core Identity.
- Application holds services and the task access policy. Infrastructure holds EF Core, the Identity mapping, and SQL Server.
- Api holds controllers, JWT, the exception handler, and the SignalR host. The React client talks to the API over HTTP only.
- Unit tests reference Domain and Application only. Integration tests reference Infrastructure and Api. Service behavior is proven by unit tests. Controller and data-access behavior is proven by integration tests. Those tests ship in the same commit as the behavior.
- Serilog on the host logs requests and errors. Register, login, create, update, delete, and import each write their own activity line. Passwords and tokens are never logged.
- There is no task-history table, no category create or delete, and no deployment phase.

Phase 1 stays on `feature/project-setup`. Each later phase starts on a new branch after the previous phase is merged.

The solution gitignore covers build output, dependencies, logs, and the whole `.cursor/` folder. It does not name individual files inside that folder.

## Phase overview

| Phase | Name | Tasks | Done when |
| --- | --- | --- | --- |
| 1 | Foundation | 01-01 to 01-05 | Solution and client scaffold build, SQL Server is defined, no feature code |
| 2 | Data model | 02-01 to 02-02 | Migration creates users, roles, categories, and tasks |
| 3 | API host | 03-01 to 03-03 | A failed request returns a safe message and the exception is logged |
| 4 | Accounts | 04-01 to 04-08 | Signup creates only a User, login opens the shell, profile checked in a browser |
| 5 | Tasks | 05-01 to 05-10 | Assignee-only vs admin-all holds, and dashboard, detail, and the form are checked in a browser |
| 6 | List, files, live updates | 06-01 to 06-05 | Filtered export and rejected import match the brief, and live updates reach only the assignee and admins |
| 7 | Submission quality | 07-01 to 07-02 | C# and JavaScript are configured for SonarQube, and the README can start the system |

## Phase 1 — Foundation

Objective: a buildable empty solution and the planning docs. No business behavior.

Depends on: nothing.

Deliverables: brief, this plan, solution, default client, SQL Server compose, gitignore.

Done when: the solution and client scaffold build, SQL Server is defined in Compose, and no feature code exists.

### PHASE-01-TASK-01 — Record the project brief

Description: Put the published assignment and the locked decisions into the repo, including the product name Anjaam.

Scope: [docs/project-brief.md](docs/project-brief.md).

Dependencies: none.

Acceptance criteria:

- The published brief and the locked decisions are present and unchanged from the file at the start of this task.
- The product name Anjaam is in the decisions.

Commit: `docs(brief): record the Shine assignment and locked decisions`

### PHASE-01-TASK-02 — Record the phase plan

Description: Add this file so later work follows one task and one commit.

Scope: [phases.md](phases.md).

Dependencies: PHASE-01-TASK-01.

Acceptance criteria:

- Every requirement in the brief maps to a task in the coverage section.

Commit: `docs(plan): add the phased development plan`

### PHASE-01-TASK-03 — Add the empty .NET 10 solution

Description: Create the projects and the reference rules. No entities, no endpoints, no feature code.

Scope: `global.json`, `TaskManagement.slnx`, Domain, Application, Infrastructure, Api, unit tests, integration tests.

Dependencies: PHASE-01-TASK-02.

Acceptance criteria:

- `dotnet build` succeeds.
- Application references Domain. Infrastructure references Application. Api references Application and Infrastructure.
- Unit tests reference Domain and Application only. Integration tests reference Infrastructure and Api.
- A unit test project cannot see the database project.
- No entities or endpoints exist.
- `global.json` pins the stable .NET 10 SDK so a release-candidate SDK is not used.

Commit: `chore(solution): add the empty .NET solution`

### PHASE-01-TASK-04 — Add the default Vite React app

Description: Add the client scaffold only. No product screens and no Anjaam layout yet.

Scope: `client/`.

Dependencies: PHASE-01-TASK-03.

Acceptance criteria:

- `npm run build` succeeds.
- No product screens exist.

Commit: `chore(client): add the default React app`

### PHASE-01-TASK-05 — Add local SQL Server and gitignore

Description: Define SQL Server for local development and ignore generated files. The database password is local-only.

Scope: `docker-compose.yml`, `.gitignore`, a short run section in `README.md`.

Dependencies: PHASE-01-TASK-03.

Acceptance criteria:

- Compose defines SQL Server.
- The README can build the solution, start the client, and start SQL.
- Gitignore covers build output, dependencies, logs, and `.cursor/` without naming the files inside `.cursor/`.
- The database password lives in development settings.

Commit: `chore(dev): add SQL Server and the solution gitignore`

## Phase 2 — Data model

Objective: the schema matches the locked task, category, user, and role rules.

Depends on: Phase 1.

Deliverables: domain types, EF mapping, seeds, initial migration.

Done when: the migration creates users, roles, categories, and tasks, and the three test layers have a place to call.

### PHASE-02-TASK-01 — Add domain types

Description: Add the task, category, status, priority, and a plain user. The creator is stored for the activity log and does not grant access.

Scope: Domain project.

Dependencies: PHASE-01-TASK-03.

Acceptance criteria:

- Task, category, status, priority, and a plain user exist.
- Domain does not reference Identity or Entity Framework.
- Title, category, and assignee are required. Description and due date are optional.
- Status is Pending, In progress, or Completed. Priority is Low, Medium, or High.
- Creator is stored and does not grant access.

Commit: `feat(domain): add task, category, and user types`

### PHASE-02-TASK-02 — Map and migrate the database

Description: Map the domain onto SQL Server, link Identity outside Domain, and seed the local data. The API does not create or delete categories. It does return the three seeded names so the task form is not hardcoded.

Scope: Infrastructure, initial migration, integration test.

Dependencies: PHASE-02-TASK-01, PHASE-01-TASK-05.

Acceptance criteria:

- EF maps the domain types. Identity is linked outside Domain.
- Work, Personal, and Study are seeded.
- One local admin is seeded from development settings: display name Local Admin, email `admin@taskmanagement.local`.
- Nothing in the API creates or deletes a category.
- One read-only list returns Work, Personal, and Study.
- The data-access test covers the seed.

Commit: `feat(persistence): map the schema and add the initial migration`

## Phase 3 — API host

Objective: the API starts, logs safely, and hides internal failures.

Depends on: Phase 2.

Deliverables: running host, Serilog, global errors.

Done when: a failed request returns a safe message and the real exception is logged.

### PHASE-03-TASK-01 — Boot the host

Description: Start the API with Swagger, health checks, and CORS for the Vite origin. Without CORS, the first login screen cannot call the API.

Scope: Api `Program`, Swagger, CORS, health.

Dependencies: PHASE-02-TASK-02.

Acceptance criteria:

- Live health returns healthy without SQL.
- Ready health checks the database.
- CORS allows the Vite dev origin.

Commit: `chore(api): boot the API host`

### PHASE-03-TASK-02 — Add Serilog

Description: Log requests and errors on the host. User actions are logged later, by the feature that performs them.

Scope: Api and Infrastructure logging.

Dependencies: PHASE-03-TASK-01.

Acceptance criteria:

- Requests and errors are logged.
- Passwords and tokens are not logged.

Commit: `feat(logging): log requests and errors`

### PHASE-03-TASK-03 — Add safe errors

Description: Return stable messages for validation, forbidden, and unexpected failures. The database text stays in the log.

Scope: Application error types, Api handler.

Dependencies: PHASE-03-TASK-02.

Acceptance criteria:

- Validation, forbidden, and unexpected failures have stable messages.
- Database text is not returned to the client.
- The exception is logged.

Commit: `feat(errors): return safe errors and log the exception`

## Phase 4 — Accounts, end to end

Objective: a person can register, sign in, and see their profile in the browser.

Depends on: Phase 3.

Deliverables: register, login, profile API, logo, login screen, signup screen, shell, profile screen.

Done when: signup creates only a User, login opens the shell, and the profile has been checked in a browser.

API tasks come before the screens that use them. Each screen also needs its own approved design. Every screen task requires an approved mockup, a browser pass at desktop, tablet, and mobile, and no known major defect.

### PHASE-04-TASK-01 — Register

Description: Create a regular user. Signup does not open a session.

Scope: Application service, Api endpoint, tests.

Dependencies: PHASE-03-TASK-03.

Acceptance criteria:

- Full name and email are required. Email is unique. Display names may be shared.
- The password needs 8 characters with upper, lower, a digit, and a symbol.
- The role is User. Signup cannot create an Admin.
- The response does not open a session.
- The registration is written to the activity log.
- Service tests and an API test ship in this commit.

Commit: `feat(auth): register a regular user`

### PHASE-04-TASK-02 — Log in

Description: Issue a 60-minute JWT and reject a missing or expired token.

Scope: token issuance, protected-route rejection, tests.

Dependencies: PHASE-04-TASK-01.

Acceptance criteria:

- Valid credentials return a 60-minute JWT. Bad credentials fail.
- A missing or expired token is rejected.
- A successful login is written to the activity log.
- Service tests and an API test ship in this commit.

Commit: `feat(auth): issue a sixty-minute JWT`

### PHASE-04-TASK-03 — Profile and account list

Description: Return the signed-in user, and give admins the account list they need to assign tasks.

Scope: current-user profile, admin-only account list, tests.

Dependencies: PHASE-04-TASK-02.

Acceptance criteria:

- The caller gets name, email, and role.
- An admin can list accounts with name, email, and role.
- A regular user receives forbidden on that list.
- Service tests and an API test ship in this commit.

Commit: `feat(auth): return the profile and the admin account list`

### PHASE-04-TASK-04 — Logo

Description: Add the approved Anjaam mark. The name is already chosen. The mark is not. Original concepts are shown first, and nothing is integrated until one is approved. No existing wordmark is copied.

Scope: the approved logo asset and its place in the client.

Dependencies: PHASE-01-TASK-04.

Acceptance criteria:

- Original concepts were shown.
- The integrated mark is the one that was approved.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(brand): add the approved logo`

### PHASE-04-TASK-05 — Login screen

Description: Build the login page and the shared API client. Later screens reuse this client. The token lives in session storage.

Scope: login page and the shared API client.

Dependencies: PHASE-04-TASK-02, PHASE-04-TASK-04.

Acceptance criteria:

- The approved login mockup is what was built.
- The token is kept in session storage.
- Success opens the shell, not a dashboard that does not exist yet.
- Later screens reuse this client.
- Desktop, tablet, and mobile have been checked in a browser, including a failed login.

Commit: `feat(client): add the login screen`

### PHASE-04-TASK-06 — Signup screen

Description: Build signup. Success returns the person to login.

Scope: signup page.

Dependencies: PHASE-04-TASK-01, PHASE-04-TASK-04.

Acceptance criteria:

- The approved signup mockup is what was built.
- Success goes to login, not the shell.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the signup screen`

### PHASE-04-TASK-07 — Application shell

Description: Add the signed-in frame. It only links to pages that already exist.

Scope: application shell.

Dependencies: PHASE-04-TASK-04, PHASE-04-TASK-05.

Acceptance criteria:

- The approved shell mockup is what was built.
- The shell shows Profile.
- It does not link to Dashboard, Tasks, or New task.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the application shell`

### PHASE-04-TASK-08 — Profile screen

Description: Show the signed-in person and let them log out. The profile does not edit name, email, or role.

Scope: profile page.

Dependencies: PHASE-04-TASK-03, PHASE-04-TASK-07.

Acceptance criteria:

- The approved profile mockup is what was built.
- Name, email, and role are shown and cannot be edited.
- Logout clears the token.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the profile screen`

## Phase 5 — Tasks, through the first screens

Objective: task rules on the server, then the screens that do not depend on import or export.

Depends on: Phase 4.

Deliverables: create, list, detail, update, delete, counts, filters, dashboard, task detail, task form.

Done when: a user only touches tasks assigned to them, an admin touches every task, and dashboard, detail, and the form have been checked in a browser.

The access policy is created with the first task operation and reused after that. Screen tasks require an approved mockup and a browser pass at desktop, tablet, and mobile.

### PHASE-05-TASK-01 — Create a task

Description: Create a task and the shared access policy. A new task starts Pending. Omitted priority is Medium.

Scope: Application access policy, create endpoint, tests.

Dependencies: PHASE-04-TASK-03.

Acceptance criteria:

- Title, category, and assignee are required. Description and due date are optional.
- Status starts at Pending. Omitted priority is Medium.
- A user can assign only themselves. An admin can assign any account, including themselves.
- Creation is logged.
- Later task operations call this same access policy.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): create a task`

### PHASE-05-TASK-02 — List visible tasks

Description: Return the caller's tasks, paged and ordered. Dashboard counts do not use this filter set; this task is the unfiltered visible list.

Scope: list endpoint, tests.

Dependencies: PHASE-05-TASK-01.

Acceptance criteria:

- A user sees only tasks assigned to them. An admin sees all.
- Page size is 20.
- Tasks with a due date come first, soonest first, then the rest by title.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): list visible tasks`

### PHASE-05-TASK-03 — Get one task

Description: Return one visible task. A hidden task looks the same as a missing one.

Scope: get-by-id endpoint, tests.

Dependencies: PHASE-05-TASK-01.

Acceptance criteria:

- The caller receives a visible task.
- A missing id and a hidden task both return not found.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): get a task by id`

### PHASE-05-TASK-04 — Update a task

Description: Edit a visible task. A due date does not move the status. A user cannot change the assignee.

Scope: update endpoint, tests.

Dependencies: PHASE-05-TASK-03.

Acceptance criteria:

- A user can edit only while the task is assigned to them. An admin can edit any task.
- A user cannot change the assignee. An admin can reassign, including to themselves.
- Any of the three statuses can be set. Nothing changes status automatically.
- The due date does not change status, including when it is in the past.
- The update is logged.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): update a task`

### PHASE-05-TASK-05 — Delete a task

Description: Remove a visible task. The activity log entry stays. There is no restore.

Scope: delete endpoint, tests.

Dependencies: PHASE-05-TASK-03.

Acceptance criteria:

- Delete follows the same visibility rule as get.
- The row is removed.
- The activity log entry remains.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): delete a task`

### PHASE-05-TASK-06 — Dashboard counts

Description: Count visible tasks by status. List filters do not change these numbers.

Scope: counts endpoint, tests.

Dependencies: PHASE-05-TASK-02.

Acceptance criteria:

- Counts are pending, in progress, and completed.
- Counts include visible tasks only.
- List filters do not change them.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): count tasks by status`

### PHASE-05-TASK-07 — Search and filters

Description: Narrow the visible list. The admin assignee filter is not available to a regular user.

Scope: list filters, tests.

Dependencies: PHASE-05-TASK-02.

Acceptance criteria:

- Text matches title and description.
- The list filters by status, priority, category, and a due-date range.
- An admin can filter by assignee. A regular user cannot.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): filter and search the task list`

### PHASE-05-TASK-08 — Dashboard screen

Description: Show the three counts. This commit is what adds the Dashboard link to the shell. After this, the dashboard is the home inside the shell.

Scope: dashboard page, shell link.

Dependencies: PHASE-05-TASK-06, PHASE-04-TASK-07.

Acceptance criteria:

- The approved dashboard mockup is what was built.
- A regular user sees their own counts. An admin sees counts for every task.
- The shell gains the Dashboard link in this commit.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the dashboard`

### PHASE-05-TASK-09 — Task detail screen

Description: Show one task. Delete lives on this screen.

Scope: task detail page.

Dependencies: PHASE-05-TASK-03, PHASE-05-TASK-05, PHASE-04-TASK-07.

Acceptance criteria:

- The approved detail mockup is what was built.
- The task is loaded by id.
- Delete is on this screen.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the task detail screen`

### PHASE-05-TASK-10 — Task form

Description: One form creates and updates. A successful create opens that task's detail, because the list does not exist yet. This commit adds the New task link to the shell.

Scope: task form, shell link.

Dependencies: PHASE-05-TASK-01, PHASE-05-TASK-04, PHASE-04-TASK-07.

Acceptance criteria:

- The approved form mockup is what was built.
- The same form creates and updates.
- A user cannot pick another assignee. An admin chooses any account.
- Category choices come from the read-only category list.
- A successful create opens that task's detail page.
- The shell gains the New task link in this commit.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the task form`

## Phase 6 — List, files, and live updates

Objective: the task list page, designed once, with export, import, and live status changes.

Depends on: Phase 5.

Deliverables: export, import, SignalR, the task list screen, and live updates on the dashboard and list.

Done when: a filtered export and a rejected import behave as locked, and a status change updates the open dashboard and list for the assignee and admins only.

The list screen requires an approved mockup and a browser pass. The live-update task is retested in the browser on the dashboard and the list. Live updates do not change the layout, so they do not need a second mockup.

### PHASE-06-TASK-01 — Export CSV

Description: Export the caller's visible tasks using the active filters. The file is not limited to the current page.

Scope: export endpoint, tests.

Dependencies: PHASE-05-TASK-07.

Acceptance criteria:

- Export uses the caller's visibility and the active list filters.
- It is not limited to the current page.
- Columns are title, description, status, priority, category, due date, and assignee email.
- This task owns the header spelling. Import uses the same headers.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): export tasks to csv`

### PHASE-06-TASK-02 — Import CSV

Description: Create tasks from CSV. The file is checked completely before any row is written.

Scope: import endpoint, tests.

Dependencies: PHASE-06-TASK-01.

Acceptance criteria:

- Columns match the export.
- An empty file or any invalid row writes nothing and returns the row errors.
- Status, priority, and category must be known values. The assignee email must belong to an existing account.
- A user can import only rows assigned to their own email. An admin can import rows for any existing account.
- Import always creates tasks. It does not update or match existing ones.
- The attempt is logged.
- Service tests and an API test ship in this commit.

Commit: `feat(tasks): import tasks from csv`

### PHASE-06-TASK-03 — Live update API

Description: Publish a task change only to the people who are allowed to see it.

Scope: SignalR hub, tests.

Dependencies: PHASE-05-TASK-04.

Acceptance criteria:

- The server sends a change only to the current assignee and to admins.
- A user cannot receive another person's task.
- Reassignment tells the previous assignee to drop the task.
- Tests for this behavior ship in this commit.

Commit: `feat(tasks): publish live task updates`

### PHASE-06-TASK-04 — Task list screen

Description: Build the list once, with filters, navigation, export, and import. This commit adds the Tasks link to the shell.

Scope: task list page, shell link.

Dependencies: PHASE-06-TASK-01, PHASE-06-TASK-02, PHASE-05-TASK-02, PHASE-05-TASK-07, PHASE-04-TASK-07.

Acceptance criteria:

- The approved list mockup is what was built.
- Filters, detail links, the new-task control, export, and import are on the page.
- A failed import shows the row errors and leaves the list unchanged.
- The shell gains the Tasks link in this commit.
- Desktop, tablet, and mobile have been checked in a browser.

Commit: `feat(client): add the task list`

### PHASE-06-TASK-05 — Live updates in the client

Description: Make the open dashboard and the open list update without a reload. The layout does not change.

Scope: dashboard and task list clients.

Dependencies: PHASE-06-TASK-03, PHASE-06-TASK-04, PHASE-05-TASK-08.

Acceptance criteria:

- The open dashboard counts and the open list update without a reload.
- A reassigned task disappears for the previous assignee.
- No second page design is required.
- The dashboard and the list are retested in a browser.

Commit: `feat(client): show live task updates`

## Phase 7 — Submission quality

Objective: a reviewer can run the app and scan it.

Depends on: Phases 1 through 6.

### PHASE-07-TASK-01 — SonarQube

Description: Configure analysis for C# and JavaScript. A missing server does not block the commit. Say so if a live scan cannot run.

Scope: scanner config for C# and JavaScript.

Dependencies: Phase 6.

Acceptance criteria:

- Both C# and JavaScript are configured.
- A live scan is reported separately if no server is available. The commit does not claim a scan ran when it did not.

Commit: `chore(quality): configure SonarQube for C# and JavaScript`

### PHASE-07-TASK-02 — Runbook

Description: Expand the README so someone else can start the system and the tests.

Scope: [README.md](README.md).

Dependencies: PHASE-07-TASK-01.

Acceptance criteria:

- SQL Server, the API, the client, and the tests can be started from the instructions.
- The runbook has been followed once.

Commit: `docs(readme): explain how to run the system`

## Coverage

| Brief requirement | Tasks |
| --- | --- |
| Registration, full name, unique email, password rules, role User | PHASE-04-TASK-01, PHASE-04-TASK-06 |
| Login, 60-minute JWT, missing or expired token rejected | PHASE-04-TASK-02, PHASE-04-TASK-05 |
| Login opens the shell; dashboard becomes home once it exists | PHASE-04-TASK-05, PHASE-05-TASK-08 |
| Admin and User roles, seeded local admin | PHASE-02-TASK-02, PHASE-04-TASK-01 |
| Admin account list | PHASE-04-TASK-03 |
| Profile and logout | PHASE-04-TASK-03, PHASE-04-TASK-08 |
| Create, read, update, delete | PHASE-05-TASK-01 through PHASE-05-TASK-05, PHASE-05-TASK-09, PHASE-05-TASK-10 |
| Assignment | PHASE-05-TASK-01, PHASE-05-TASK-04, PHASE-05-TASK-10 |
| Priority, category, due date | PHASE-02-TASK-01, PHASE-02-TASK-02, PHASE-05-TASK-01, PHASE-05-TASK-04 |
| Visibility, hidden task returns not found | PHASE-05-TASK-01 through PHASE-05-TASK-05 |
| Serilog requests, errors, and exceptions | PHASE-03-TASK-02, PHASE-03-TASK-03 |
| User activity | PHASE-04-TASK-01, PHASE-04-TASK-02, PHASE-05-TASK-01, PHASE-05-TASK-04, PHASE-05-TASK-05, PHASE-06-TASK-02 |
| SQL Server and Entity Framework | PHASE-01-TASK-05, PHASE-02-TASK-02 |
| xUnit for services, controllers, and data access | PHASE-02-TASK-02 and the API tasks in Phases 4, 5, and 6 |
| SonarQube for C# and JavaScript | PHASE-07-TASK-01 |
| React screens | PHASE-04-TASK-04 through PHASE-04-TASK-08, PHASE-05-TASK-08 through PHASE-05-TASK-10, PHASE-06-TASK-04 |
| Git branches, no direct commits to main | This file, Phase 1 on `feature/project-setup`, a new branch per later phase |
| SignalR | PHASE-06-TASK-03, PHASE-06-TASK-05 |
| Import and export | PHASE-06-TASK-01, PHASE-06-TASK-02, PHASE-06-TASK-04 |
| Search and filters | PHASE-05-TASK-07, PHASE-06-TASK-04 |
| Dashboard counts | PHASE-05-TASK-06, PHASE-05-TASK-08 |
| Product name Anjaam | This file and the brief. The logo is PHASE-04-TASK-04 |

No task creates or deletes categories. No task adds a task-history table. No task deploys to a server.

## Execution order

01-01, 01-02, 01-03, then 01-04 and 01-05 after 01-03, then 02-01, 02-02, 03-01, 03-02, 03-03, then Phase 4 in task order, then Phase 5 in task order, then Phase 6 in task order, then 07-01, 07-02.

PHASE-04-TASK-04 can start as soon as the client scaffold exists. It still has to finish before the login screen, because the login screen depends on the approved logo.

## Completion criteria

The project is complete when every task's acceptance criteria are met, the coverage list has no missing brief requirement, and the README runbook has been followed once.

## Open questions

These do not block PHASE-01-TASK-01 or PHASE-01-TASK-02. Each one has to be settled before the task that uses it.

- The seeded admin is Local Admin, `admin@taskmanagement.local`. The password stays in development settings. Change this before PHASE-02-TASK-02 if a different account is wanted.
- The React app keeps the JWT in session storage. It is cleared on logout and when the browser session ends. Change this before PHASE-04-TASK-05 if a different store is wanted.
- CSV column order is title, description, status, priority, category, due date, and assignee email. PHASE-06-TASK-01 owns the header spelling, and import uses those same headers.
- SonarQube configuration does not require a running server. A scan is reported as blocked if no server exists.

The product name is settled. It is Anjaam.
