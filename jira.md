# Jira Masterclass

> Jira is a **Project Tracking / Project Management Tool** — used to plan, track, and manage software development work (bugs, features, tasks) across a team.

---

## What is Agile?

> Agile is a way of building software in small, incremental chunks — plan a bit, build a bit, test a bit, get feedback, repeat — instead of trying to plan and build the entire product upfront. It prioritizes flexibility, fast feedback, and adapting to change over rigid, long-term planning.

## Methodologies

* **Waterfall** — the traditional approach: fully plan → then design → then build → then test → then deploy, all in strict sequence, one phase completed before the next starts. Very rigid, hard to change direction mid-project.

Other relevant methodologies (beyond Waterfall vs Agile):
* **Scrum** — Agile methodology built around fixed-length iterations called **Sprints**, with defined roles (Scrum Master, Product Owner) and ceremonies (standups, sprint planning, retrospectives). The most widely used Agile framework in software teams.
* **Kanban** — Agile methodology focused on **continuous flow** of work through visual columns (To Do → In Progress → Done) rather than fixed sprints — work items move whenever they're ready, no fixed iteration length.
* **Lean** — focuses on maximizing value while minimizing waste (unnecessary steps, delays, excess work) — often the underlying philosophy that both Kanban and Agile draw from.

**In DevOps specifically**, the practical cycle is usually described as:
> **Code → Build → Test → Deploy** (often looped continuously via CI/CD — this is the DevOps take on "agile," focused on shipping fast and often.)

---

## Scrum

> A structured Agile framework where work is broken into fixed time-boxed iterations called **Sprints** (commonly 1–2 weeks), with regular check-ins and a clear set of roles/rituals to keep the team aligned.

### Scrum Master

> Not a traditional "manager" — the Scrum Master is a facilitator/coach whose job is to help the team follow the Scrum process smoothly, remove blockers, run ceremonies (standups, planning, retros), and shield the team from distractions so they can focus on delivering the sprint's work.

### Standup (Daily Standup / Daily Scrum)

> A short (usually 10-15 min), daily team meeting where each person answers:
1. What did I do yesterday?
2. What am I doing today?
3. Any blockers?

Purpose: quick visibility and early flagging of issues — not a status report to management, more a sync between teammates.

### Common Task States

```
To Do → In Progress → Testing → Done
```
Each ticket/task moves left to right across a board as work progresses. (Teams often customize this — e.g. adding "In Review," "Blocked," etc.)

---

## Kanban vs Scrum

| | Scrum | Kanban |
|---|---|---|
| Work structure | Fixed-length **Sprints** (e.g. 2 weeks) | Continuous flow, no fixed iterations |
| Planning | Planned upfront per sprint, then locked | Work pulled in as capacity allows, ongoing |
| Roles | Defined roles (Scrum Master, Product Owner) | No mandatory defined roles |
| Best for | Teams that want predictable, regular delivery cycles | Teams with unpredictable/continuous incoming work (e.g. support/ops teams) |
| Board reset | Board typically resets each new sprint | Board is persistent/ongoing |

---

## Sprints

> A **Sprint** is a fixed, time-boxed period (commonly 1–2 weeks) during which a Scrum team commits to completing a specific set of work. At the end, there's usually a **Sprint Review** (demo what was built) and a **Retrospective** (what went well/what to improve), then the next sprint begins.

---

## Epic, Story, Task — the Hierarchy

```
Epic   → the big-picture GOAL
  └── Story → what needs to be achieved, from a user/role's perspective, to reach that goal
        └── Task → the specific, concrete steps to actually get that story done
```

* **Epic** → the goal. E.g. *"Migrate infrastructure to AWS."*
* **Story** → framed from a role's perspective, describing what they need to achieve. E.g. *"As a DevOps engineer, I need the app servers running on EC2 so we're off the old data center."*
* **Task** → the concrete how. E.g. *"Set up EC2 instances," "Configure security groups," "Write deployment script."*

(Your phrasing was already right — Epic = goal, Story = what I must achieve, Task = how I'll achieve it.)

---

## Story Points

> A relative measure of **effort/complexity/uncertainty** for a piece of work — deliberately **not** a direct measure of hours, though it does roughly correlate to how much time/effort the team expects it'll take.

**Common scale (Fibonacci-like):** `1, 2, 3, 5, 8, 13`

| Points | Rough meaning |
|---|---|
| **1** | Trivial — very small, quick, well understood (few hours or less) |
| **2** | Small — straightforward, minor complexity |
| **3** | Small-medium — a bit more involved but still clear |
| **5** | Medium — noticeable effort, some moving parts or minor unknowns |
| **8** | Large — significant effort, real complexity or some uncertainty |
| **13** | Very large — high complexity/uncertainty; often a sign it should be broken down into smaller stories |

**Why Fibonacci-style spacing (not 1,2,3,4,5...)?** Because as effort/complexity grows, precision naturally gets fuzzier — it's easy to tell a 1-point task from a 2-point task, but much harder to precisely distinguish an 8 from a 9. The bigger jumps force the team to round to the nearest "bucket" instead of debating pointless precision.

---

## Common Jira Interview Questions

* What's the difference between an Epic, Story, and Task?
* What's the difference between Scrum and Kanban?
* What is a Sprint, and what happens during Sprint Planning / Sprint Review / Retrospective?
* What are Story Points, and how are they estimated (e.g. Planning Poker)?
* What's the role of a Scrum Master vs a Product Owner?
* How do you handle a ticket that's blocked?
* Have you used Jira automation rules? Give an example.
* How would you integrate Jira with Slack/GitHub in a real workflow?
* What's the difference between a Bug and a Task in Jira?
* How do you prioritize a backlog?

---

## Integrating Slack with Jira

1. In Jira, go to **Apps → Explore more apps** (or from Slack: **Apps → search "Jira Cloud"**).
2. Install the official **"Jira Cloud" app for Slack** (or vice versa — install from either side, they link the same way).
3. Authenticate/connect your Jira account and Slack workspace when prompted.
4. Once connected, in any Slack channel you can:
   * Run `/jira connect` to link that channel.
   * Get **automatic notifications** in the channel when a ticket is created, updated, commented on, or its status changes.
   * Create/view/comment on Jira tickets directly from Slack using slash commands (e.g. `/jira create`).

**Real use:** a team channel gets a Slack message the moment a critical bug ticket is created or moved to "In Progress," so everyone sees it instantly without needing to check Jira manually.

---

## Creating Automations in Jira (Slack + GitHub + one more)

Jira has a built-in **Automation** feature (Project Settings → Automation → Create Rule) that works on a simple **Trigger → Condition → Action** structure — no coding required.

### Example 1 — Jira + Slack automation
* **Trigger:** Issue status changes to "In Review"
* **Condition:** (optional) only for issues in a specific project
* **Action:** Send a Slack message to a specific channel — e.g. *"PROJ-123 is now In Review, please check it out."*

### Example 2 — Jira + GitHub automation
* **Trigger:** A GitHub PR linked to a Jira ticket gets merged (via the GitHub-for-Jira integration)
* **Action:** Automatically transition the Jira ticket's status to "Done," and add a comment linking the merged PR.

### Example 3 — Jira + Email (or Microsoft Teams) automation
* **Trigger:** A new issue is created with priority = "Highest"
* **Action:** Send an email notification (or a Microsoft Teams message, another common one alongside Slack) to the team lead immediately — useful for critical/urgent bug alerts that shouldn't wait for someone to check the board.

**Why this matters in real teams:** automations remove manual busywork — no one has to remember to post a Slack update or manually flip a ticket's status after a merge. It all just happens based on the rules you set once.
