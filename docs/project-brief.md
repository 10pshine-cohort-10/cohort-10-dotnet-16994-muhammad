# Task Management Tool

Shine candidate portal brief for Muhammad Bilal. The sections through "Not specified in the brief" record the assignment as published. The decisions after that are the choices we locked for this repo.

## Assignment

| Item | Detail |
| --- | --- |
| Program | 10Pearls Shine, Cohort 10 |
| Portal user | Bilal12 |
| Candidate | Muhammad Bilal |
| University | FAST National University of Computer and Emerging Sciences (Karachi Campus) |
| CGPA | 3.58 |
| Domain | .NET Fullstack (.NET + ReactJS) |
| Project | Task Management Tool |
| Status | Pending |
| Due | 11 January 2027, 11:59 PM |
| Noted on | 14 November 2026, when the portal showed 59 days remaining |

Repository: `cohort-10-dotnet-16994-muhammad`. Current work for this brief stays on `feature/project-setup`. `main` is not used for direct commits.

## Overview

A web-based task management system. Users organize and track tasks. The system must support authentication and authorization, task CRUD, application logging, exception handling, unit tests, and a SQL database.

The build aim, stated for this repo, is the full portal scope, including the features the portal marks optional.

## Technology stack

Required:

- ASP.NET Core
- React.js
- Entity Framework
- SQL Server
- Serilog
- xUnit
- SonarQube
- Git

## Required features

### User authentication and authorization

- User registration and login.
- Roles, with admin and regular user given as examples.
- Access restricted by role.

### Task management

- Create, read, update, and delete tasks.
- Assign tasks to users.
- Set task priorities.
- Categorize tasks.
- Add due dates.

### Application logging with Serilog

- Logging throughout the application.
- Log important events, errors, and user activities.

### Entity Framework and SQL database

- Entity Framework for data access.
- A SQL schema for user information, tasks, and related data.

### Exception handling

- Global exception handling.
- Meaningful error messages for users.
- Exceptions logged with Serilog.

### Unit testing with xUnit

- Tests for critical parts of the application.
- Tests for controllers, services, and the data access layer.

### SonarQube

- Analyze code quality and report issues.
- Rules configured for C# and JavaScript.

### React frontend

- An interactive, responsive interface.
- A dashboard for tasks, the user profile, and other relevant information.

### Git

- Version control.
- Branching and merging rather than committing directly to `main`.

## Additional features

The portal lists these as optional. They are in scope for this build.

- Real-time task status updates with SignalR.
- Export tasks to a file and import tasks from a file.
- Search and filter for tasks. The task list screen also requires filters.

## Application screens

### Sign up / log in

Components: signup form, login form.

Operations:

- Register a user and authenticate a user.
- Redirect to the dashboard after a successful login.

### Dashboard

Components: completed count, in-progress count, pending count.

Operations:

- For a regular user, return that user's task counts by status.
- For an admin or super user, return counts for all tasks by status.

### Task list

Components: a task list with filters, a link to each task's detail page, and a link or button to create a task.

Operations:

- Fetch the user's task list from the backend.
- Display the list.
- The new-task control navigates to the new-task screen.

### Task detail

Components: task detail.

Operations:

- Fetch the task by id from the backend.
- Display the task.

### New task

Components: a form with the required controls.

Operations:

- Read the inputs and post them to the backend to create or update a task.

### User profile

Components: user detail, logout button.

Operations:

- Get the user from the backend and display the detail.
- Log the user out.

## Not specified in the brief

The portal leaves these open. The choices we are using are in the next section.

- Whether "super user" is a third role or another name for admin.
- How a caller is authenticated.
- The allowed priority values.
- Whether a category is its own record or free text.
- Which statuses exist beyond Pending, InProgress, and Completed.
- Who may assign, view, edit, or delete a task.
- The file format for import and export.
- Which filters the task list offers.
- Where SQL Server runs locally.

## Decisions

Locked for this build.

### Product

The product name is Anjaam (انجام). It means outcome or completion. The line under the name is "Bring every task to completion." The assignment title in the README stays Task Management Tool so the submission matches the portal. The logo is designed and approved later. It is not chosen in this document.

### Roles

Two roles: Admin and User. "Super user" in the dashboard means Admin. There is no third role.

Signup creates a User. It cannot create an Admin. One Admin account is seeded for local development, and its password stays in development settings. After signup the person signs in. A successful login opens the application shell. Once the dashboard exists, it is the home inside that shell. Signup itself never opens a session.

### Visibility

A regular user sees tasks assigned to them. An admin sees every task. The creator is stored for the activity log and does not grant access. After an admin assigns a task to someone else, the previous assignee loses it.

Counts, list, detail, update, delete, and export all use this rule. Dashboard counts ignore list filters. A missing task, and a task the caller cannot see, both return not found.

### Assignment

Every task has one assignee. A user can only assign a task to themselves, on create and on update. An admin must choose an account and may assign a task to any account, including themselves. The admin needs a list of accounts (name, email, role) to do that. A regular user does not receive that list.

### Edit and delete

A user may edit or delete a task only while it is assigned to them. An admin may edit or delete any task. Delete is a control on the task detail screen and removes the task. The Serilog activity entry remains. There is no restore.

A user cannot change the assignee. An admin can.

### Fields

Title, category, and assignee are required. Description and due date are optional. A due date does not change the status, including when it is in the past.

### Status

Pending, In progress, and Completed. A new task starts as Pending. Create and update may set any of the three. Nothing moves a task between them automatically.

### Priority

Low, Medium, and High. Medium is the default when the caller omits it.

### Category

A category is its own table, not free text. The seed values are Work, Personal, and Study. The app does not create or delete categories.

### Login

ASP.NET Core Identity issues a JWT that lives for 60 minutes. The React app stores that token and sends it on later requests, including SignalR. Logout deletes the token from the client. A password needs 8 characters, upper and lower case, a digit, and a symbol. Email is unique. Display names may be shared.

The profile screen shows the signed-in user's name, email, and role. It does not change them.

### Import and export

CSV, one row per task. Columns: title, description, status, priority, category, due date, and assignee email. Export uses the caller's visibility and the active list filters, and it is not limited to the current page.

Import checks every row before it writes. If any row is invalid, it writes nothing and returns the row errors. An empty file is invalid. A known status, priority, and category are required. The assignee email must belong to an existing account. A user may import only rows assigned to their own email. An admin may import rows for any account. Import always creates tasks. It does not update or match existing ones.

### Filters

Text matches title and description. The list also filters by status, priority, category, and a due-date range. An admin can also filter by assignee account. A regular user cannot. Tasks with a due date come first, soonest first, and the rest follow by title. The list is paged at 20 tasks.

### Real-time updates

A task change is sent only to the current assignee and to admins. Reassignment also tells the previous assignee to drop the task. Other users receive nothing.

### Database

SQL Server runs in Docker on this machine. The local password stays in development settings.
