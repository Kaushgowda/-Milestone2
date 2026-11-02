# FranchiseOps AI — Milestone 2
**Infosys Springboard Internship 7.0 · Batch 4 · Author: Kaushik B**
FastAPI · SQLite · JWT · bcrypt · RBAC · Google Colab · ngrok

FranchiseOps AI is a multi-user franchise-operations platform. Six background agents watch the workforce, outlets, inventory, marketing, customer sentiment and audits, and write their actions as tickets. An AI Copilot answers questions with exact SQL. Milestone 2 turns the provided single-administrator app into a real multi-user product: login, profile, five-role access control, an Admin dashboard, a stronger Copilot and a better design.

---
## 1. Contents
1. [What was added in Milestone 2](#2-what-was-added-in-milestone-2)
2. [How to run](#3-how-to-run)
3. [Project structure](#4-project-structure)
4. [Authentication (Step 3)](#5-authentication-step-3)
5. [Profile page (Step 4)](#6-profile-page-step-4)
6. [RBAC (Step 5)](#7-rbac-step-5)
7. [Admin dashboard (Step 6)](#8-admin-dashboard-step-6)
8. [AI Copilot (Step 7)](#9-ai-copilot-step-7)
9. [The six agents](#10-the-six-agents)
10. [Design (Step 8)](#11-design-step-8)
11. [API reference](#12-api-reference)
12. [Database](#13-database)
13. [Testing](#14-testing)
14. [Troubleshooting](#15-troubleshooting)
15. [Known limits](#16-known-limits)
16. [Submission checklist](#17-submission-checklist)

---
## 2. What was added in Milestone 2
| Step | Feature | Where |
|---|---|---|
| 3 | Sign up, sign in, forgot password (security question or e-mailed OTP), bcrypt hashes, JWT, lock after 5 wrong passwords, Sign out | `auth.py`, `static/app.js` |
| 4 | Profile page: details, change username, change password | `adminpanel.py` |
| 5 | Five roles, page matrix, capabilities, outlet scope, 403 on the server | `rbac.py`, `server.py`, `views_core.py` |
| 6 | Admin dashboard: users table, add, delete, promote/demote, lock/unlock, activity log, KPI cards | `adminpanel.py`, `static/app.js` |
| 7 | 28 new Copilot SQL intents, spelling tolerance, RBAC-aware answers, SQL and row count shown, polite fallback | `analyst.py` (appended), `server.py` |
| 8 | Light theme by default with a dark switch, readable tables, mobile layout, loading and error states, 403 page | `static/style.css`, `static/app.js` |

`localuser.py` (the single local administrator) was removed.

---
## 3. How to run
1. Open the notebook in Google Colab and choose **Runtime → Run all**.
2. Add these **Colab Secrets** (key icon in the left sidebar). Never type a secret in a cell.

| Secret | Used for |
|---|---|
| `JWT_SECRET` | Signs login tokens. If missing, the app creates a random one and stores it for the session. |
| `EMAIL_ADDRESS` | Sender Gmail for OTP mails (Milestone 1). |
| `EMAIL_PASSWORD` | Gmail **app password** (not your normal password). |
| `NGROK_AUTHTOKEN` | Fallback public link if Colab blocks the first one. |

3. Wait for `FranchiseOps AI is running`, then click **Open the app**.
4. Optional: run the last cell (role self-test) to print allowed / denied (403) results for every role.

Optional secrets: `ADMIN_EMAIL_ID`, `ADMIN_PASSWORD` (override the main admin account), `PORTAL_ADMIN_USERNAME`, `PORTAL_ADMIN_PASSWORD` (override the built-in recovery admin).

### Demo accounts (testing only)
| Role | Sign in with | Password |
|---|---|---|
| Admin | **Administrator Login** → username `admin` | `Admin@123` |
| Built-in recovery admin | **Administrator Login** → username `infosys_admin` | `infosys@123` |
| Franchise Owner | `owner@infosys.com` | `Demo@1234` |
| Store Manager | `manager@infosys.com` (works at the first outlet in the data) | `Demo@1234` |
| Staff | `staff@infosys.com` (works at the first outlet in the data) | `Demo@1234` |
| Auditor | `auditor@infosys.com` | `Demo@1234` |

These are demo credentials written in the code. Change them before deploying anywhere real. To switch accounts, click **Sign out** and sign in again.

---
## 4. Project structure
The notebook writes the app to `/content/app` with `%%writefile` cells, then starts it on port 8000.

| File | Purpose |
|---|---|
| `server.py` | FastAPI app. Contains the four places where identity and permission enter the system: `user_of`, `ctx`, `need`, `menu_for`. Also the page router, detail drawer, live pulse stream and Copilot route. |
| `auth.py` | Milestone 1 login: sign up, sign in, forgot password, JWT, lockout, last login. |
| `rbac.py` | **One place for access rules:** `ROLE_MENU`, `ROLE_CAPS`, outlet scope, Copilot topics, permission matrix. |
| `adminpanel.py` | Profile page, Admin dashboard and their API routes. |
| `analyst.py`, `qa.py`, `intent_router.py` | SQL Copilot: intents, whitelisted queries, spelling fix, role guards. |
| `views_core.py`, `views_franchise.py` | Build each page from blocks (KPIs, tables, charts). |
| `blocks.py` | Block helpers (`page`, `kpis`, `table`, `form`, `callout`). |
| `live_engine.py`, `worker.py`, `domain_franchise.py` | Background worker that runs the agent cycles and writes tickets. |
| `hardening.py` | Per-IP rate limits and security headers. |
| `db.py`, `seed_data.py`, `scale_data.py` | Schema, demo accounts and demo data. |
| `static/app.js`, `style.css`, `index.html` | Single-page front end. |

---
## 5. Authentication (Step 3)
**Flow:** sign up → sign in → the server sets a signed token → every request is checked against the database.

- **Passwords** are hashed with bcrypt. A plain password is never stored, logged or returned.
- **JWT** is signed with `JWT_SECRET` (HS256) and lasts 2 hours. The browser keeps it in an **HttpOnly** cookie (`fo_session`, `SameSite=Lax`, `Secure` on https), so page scripts cannot read it. API clients may send `Authorization: Bearer <token>` instead.
- **Checked on every request:** `user_of()` reads the token, then re-reads the user from the database. A user who is **locked or deleted** is rejected on the very next request, and the live stream disconnects. A role change also applies immediately.
- **Same error text** for an unknown e-mail and a wrong password (`Invalid credentials.`). An unknown e-mail still runs one bcrypt check so response time does not reveal whether an account exists.
- **Lockout:** 5 wrong passwords lock the account. Only an Admin can unlock it. If the password is right but the account is locked, the user is told to contact an administrator. The built-in `infosys_admin` account is never locked, so it is always a way back in.
- **Validation (Milestone 1 rules):** the e-mail needs at least 2 characters before `@`, 2 between `@` and the dot, 2 after. The password needs 8+ characters with upper case, lower case, a number and a special character. Confirm password must match.
- **Forgot password:** by security question, or by a 6-digit OTP sent from `EMAIL_ADDRESS` (expires in 5 minutes, max 5 tries, then 10 minutes to choose a new password).
- **Rate limits** (`hardening.py`): login 10 per minute, sign-up 6 per 10 minutes, OTP 5 per 5 minutes, reset 8 per 10 minutes, password change 8 per 10 minutes (per IP). The notebook launcher sets `RATE_LIMIT_OFF=1` for easy testing in Colab; remove that line to switch limits on.

Self sign-up creates a **Staff** account. An Admin can change the role later.

---
## 6. Profile page (Step 4)
Every role has a **Profile** page showing username, e-mail, role, outlet (Store Manager and Staff), joined date, last login and the pages the role may open.

- **Change username:** 3–30 characters (letters, numbers, space, `.`, `-`, `_`), must be unique.
- **Change password:** the current password is asked first; then the Milestone 1 rules apply and the new password must differ from the old one.
- The password hash is never displayed. The built-in Administrator Login keeps its name and password (they come from Colab Secrets).

---
## 7. RBAC (Step 5)
Rules live in `rbac.py` and are enforced **on the server**. Hiding a menu item is not the security; the checks are.

### Page access matrix
| Role | Copilot | Workf. | Outlets | Invent. | Mktg | Sentim. | Audit | Run agents | Users | Outlet scope |
|---|---|---|---|---|---|---|---|---|---|---|
| Admin | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | all |
| Franchise Owner | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | all |
| Store Manager | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ | own outlet |
| Staff | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | own outlet |
| Auditor | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | all, read-only |

Every role also has **Profile**. The same table is shown in the Admin dashboard, computed from `ROLE_MENU` and `ROLE_CAPS` so it cannot disagree with the enforcement.

### Capabilities (`ROLE_CAPS`)
| Capability | Meaning | Roles |
|---|---|---|
| `copilot` | Use the Copilot | all |
| `view` | Open agent pages and outlet details | Admin, Franchise Owner, Store Manager, Auditor |
| `run` | Run agents | Admin, Franchise Owner |
| `resolve` | Log an inspection | Admin, Franchise Owner |
| `review` | Add a customer review | Admin, Franchise Owner, Store Manager |
| `users` | Admin dashboard and user management | Admin |

### How it is enforced
| Mechanism | Behaviour |
|---|---|
| `menu_for(user)` | Returns only the pages the role may open. |
| `GET /api/page/<id>` | **403** for any page outside the role, even when typed by hand. Unknown ids return 404. |
| `need(ctx, capability)` | Raises **403** when the role lacks the capability. Called in every action route (run agents, log inspection, add review, admin routes). |
| `ctx["scope"]` | The one outlet a Store Manager or Staff member may see. Pages, tables, tickets, outlet drawer, live counters and Copilot queries are all filtered by it. A scoped user without an outlet sees nothing (fail closed). |
| Detail drawer | `/api/detail/<outlet>` returns 403 for another outlet. |
| Write routes | Reviews and inspections for another outlet are refused with 403. |
| Live pulse | Scoped roles get counts for their own outlet only; Staff get no health or ticket numbers. |

The Auditor is read-only: no running agents, no inspections, no reviews.

---
## 8. Admin dashboard (Step 6)
Only the Admin role can open it, and every admin route calls `need(ctx, "users")`.

| Feature | Details |
|---|---|
| KPI cards | Total users, users per role, locked users. |
| Users table | Email, username, role, outlet, status (Active / Locked), created, last login. Search box. Never shows passwords or hashes. |
| Add user | Email, username, temporary password, role, outlet. Same validation as sign-up. A Store Manager or Staff user must be given an existing outlet. |
| Promote / demote | Role drop-down. The outlet drop-down is used for Store Manager and Staff. |
| Lock / Unlock | A locked user cannot sign in, and their current session stops working immediately. Unlock also resets the wrong-password counter. |
| Delete | Two-step confirmation (the button asks "Delete this user?"), and the server also requires `confirm: true`. |
| Permission table | The page access matrix above. |
| Activity log | Who, what, when, target and detail for each admin action, latest 50 rows. Automatic lockouts are logged as `system`. |

**Safety rules (enforced on the server):**
- An admin cannot delete, lock, or change the role of **their own** account.
- The **last remaining Admin** can never be deleted, locked or demoted.
- The built-in Administrator account (`infosys@ai`, behind the `infosys_admin` login) cannot be deleted, locked or demoted.
- Passwords and hashes never leave the server.

---
## 9. AI Copilot (Step 7)
The Copilot has no LLM and no RAG. **Every answer comes from one read-only SQL query**, and numbers are never made up.

### Pipeline
1. **Greeting / help / thanks** are answered directly and personalised with the user's name, role and outlet.
2. **Spelling fix:** misspelled words are matched to the domain vocabulary (`negitive reveiws` → `negative review`, `inventry`, `campain`, `lowst satisfiction`).
3. **Role guard (before any query):** the role must be allowed to read the topic, and a scoped user may not ask about another outlet.
4. **Different wordings** are mapped to one meaning (`shops`, `branches` → outlet; `employees` → staff; `CSAT` → satisfaction; `running low` → below reorder; `sales` → revenue).
5. **Intent match → SQL.** If an intent matches, its query runs (scoped to the user's outlet when needed). Otherwise the original engines run.
6. **SQL guard (after the query):** an answer is dropped if its SQL read a table the role may not see.
7. **Reply** shows the answer, a result table, **Rows returned** and the **SQL used**. If nothing matches, the Copilot replies politely and suggests **three valid questions** chosen for the role.

### Topics each role may ask about
| Role | Topics |
|---|---|
| Admin, Franchise Owner, Auditor | outlets, staff, inventory, marketing, feedback, audits, tickets |
| Store Manager | everything except marketing, only for their own outlet |
| Staff | outlets, inventory, feedback, only for their own outlet |

### The 28 new intents (`analyst.py`)
| Area | Example question |
|---|---|
| Outlets | Which outlet has the lowest CSAT? · Which outlet has the highest customer satisfaction? · Top 5 outlets by revenue · Which outlet has the lowest revenue? · Which outlets are the most profitable? · How many outlets are there? |
| Workforce | How many staff does OUT-010 have? · Which outlets have the highest attrition? · Which employees have high attrition risk? · What is the average attrition? · Who has the most overtime? |
| Inventory | List items below reorder level · Which items have the highest stock-out risk? · Which outlet has the lowest stock cover? |
| Marketing | Which campaign has the best ROI? · Which campaign has the lowest ROI? · Which channel has the best ROI? · Which campaign has the most conversions? · What is the marketing budget by channel? |
| Sentiment | Show negative reviews this week · Which outlet has the most negative feedback? · Show negative reviews · What is the average rating by outlet? · Show the lowest-rated reviews |
| Audit | Which audits need action? · Show pending compliance issues · Which outlets have the lowest audit scores? |
| Tickets | How many open tickets are there by severity? |

The original provided questions (outlet risk, explain an outlet, stock status, sales trend and so on) still work.

### Tested questions
Answers come from the generated demo data, so the values change between runs. These were checked on a test run as Franchise Owner:
| Question | Result |
|---|---|
| which outlet has the lowest CSAT | one ranked table, lowest first (example: OUT-110, 2.95) |
| how many staff does OUT-010 have | one row with the staff count |
| list items below reorder level | items with `current_stock <= reorder_threshold` |
| which campaign has the best ROI | top 10 campaigns by ROI |
| show negative reviews this week | up to 20 reviews from the last 7 days; if none, it says so and shows the latest negative reviews instead |
| show negitive reveiws this week | same as above (spelling fixed) |
| whcih outlet has the lowst satisfiction | same as the CSAT question |
| how many outlets are there | outlet count |
| which outlet has the lowest stock cover | outlets ranked by average days of cover |
| tell me a joke | polite refusal plus 3 valid suggestions |

As **Store Manager**: the lowest-CSAT question returns only your own outlet; the campaign question is refused (no marketing access); "how many staff does OUT-005 have" says it is outside your scope.
As **Staff**: staff, salary and "why is OUT-001 flagged" questions are refused.

---
## 10. The six agents
They come with the provided app and run on a background worker, writing their actions as tickets (no approval queue).

| Agent | Page | What it does |
|---|---|---|
| 1 · Workforce | `workforce` | Staffing and attrition |
| 2 · Outlets | `outlets` | Health score and ranking of every outlet |
| 3 · Inventory | `inventory` | Stock cover and reorder tickets |
| 4 · Marketing | `marketing` | Campaign performance |
| 5 · Sentiment | `sentiment` | Customer feedback sentiment |
| 6 · Audit | `audit` | Compliance checks and findings |

Admin and Franchise Owner see a **Run** button on these pages. Other roles see the data without it.

---
## 11. Design (Step 8)
- **Theme:** consistent colour tokens, **light by default**, with a 🌙 / ☀️ switch in the top bar that is remembered in the browser.
- **Tables:** zebra rows, hover highlight, scrolling on small screens.
- **Responsive:** layouts adapt for tablets and phones.
- **Loading and errors:** spinner while a page loads, "Querying the database…" in the Copilot, clear network and session-ended messages, and a dedicated **Access denied (403)** page.
- **Pages designed with extra care:** login, sign-up, forgot password, profile and Admin dashboard.

---
## 12. API reference
| Method | Path | Who | Purpose |
|---|---|---|---|
| POST | `/api/register` | public | Sign up (role Staff) |
| POST | `/api/login` | public | Sign in with e-mail and password |
| POST | `/api/login/admin` | public | Administrator Login |
| POST | `/api/reset/question`, `/api/reset/security`, `/api/reset/otp` | public | Forgot password by security question |
| POST | `/api/otp/send`, `/api/otp/verify` | public | Forgot password by e-mailed OTP |
| POST | `/api/logout` | signed in | Sign out |
| GET | `/api/me` | signed in | User, menu, capabilities, scope |
| GET | `/api/page/{id}` | role-checked | Page blocks (403 outside the role) |
| GET | `/api/detail/{id}` | `view` + scope | Outlet detail |
| POST | `/api/copilot` | `copilot` | Ask the Copilot (streamed) |
| POST | `/api/run_cycle`, `/api/run_agent` | `run` | Run agents |
| POST | `/api/franchise/audit` | `resolve` + scope | Log an inspection |
| POST | `/api/franchise/review` | `review` + scope | Add a customer review |
| POST | `/api/profile/username`, `/api/profile/password` | signed in | Change own username / password |
| POST | `/api/admin/users` | `users` | Add user |
| POST | `/api/admin/users/role`, `/outlet`, `/lock`, `/delete` | `users` | Promote/demote, set outlet, lock/unlock, delete |

---
## 13. Database
SQLite file in the app's data folder (never commit `.db` files).

| Table | Notes |
|---|---|
| `users` | email, username, `password_hash`, role, **outlet**, account status, failed attempts, created, **last login**, security question/answer (hashed) |
| `admin_activity` | id, time (UTC), actor, action, target, detail |
| `password_otps` | hashed OTPs with expiry and tries |
| `outlets`, `staff`, `inventory`, `marketing`, `feedback`, `audits` | Franchise data. Each has `outlet_id`, which is what scope filters on. |
| `agent_actions` | Tickets written by the agents |

---
## 14. Testing
The notebook's last cell signs in as every role and prints a table of pages and actions (OK or 403). Checks done on a local copy of the app:

| Area | Result |
|---|---|
| Page matrix for all 5 roles | Matches the PDF matrix exactly |
| Wrong password ×5 | Account locked; same message as unknown e-mail; right password on a locked account → "contact an administrator" |
| Lock while signed in | Next request fails immediately |
| Add user | Weak password, bad e-mail, duplicate and missing outlet are all rejected |
| Self delete / self lock / self role change | Refused |
| Last remaining Admin | Cannot be deleted, locked or demoted |
| Delete without confirmation | Refused |
| Store Manager scope | Other outlet's detail → 403; review for another outlet → 403; pages show only their outlet |
| Run agents as Store Manager, Staff, Auditor | 403 |
| Profile | Short username, wrong current password and weak new password rejected; valid change works; no hash in any response |
| Copilot | All 28 intents answered with SQL and row count; typos fixed; Store Manager and Staff topic limits enforced; off-topic question → 3 suggestions |
| Browser-style check | Login, Admin table, two-step delete, 403 page for Staff and Store Manager, theme switch |

To test by hand: sign in as each role, open each page, try each action, and screenshot one allowed and one denied (403) case.

---
## 15. Troubleshooting
| Problem | Fix |
|---|---|
| The "Open the app" link is blocked | Add `NGROK_AUTHTOKEN` to Colab Secrets and run the last launcher cell again. |
| No OTP e-mail | Check `EMAIL_ADDRESS` and that `EMAIL_PASSWORD` is a Gmail **app password**. Use the security question instead. |
| "Please use Administrator Login" | Admin accounts sign in through the **Administrator Login** button, not the normal form. |
| Account locked | An Admin unlocks it in the dashboard. If every admin is locked, sign in with `infosys_admin`. |
| Pages look empty right after start | The worker needs a minute to run its first agent cycle. Wait, then refresh. |
| The app opens in dark mode | Click the ☀️ button once; the choice is saved. |
| Edited a cell but nothing changed | Restart the app (re-run the launcher cell); the server does not reload code by itself. |

---
## 16. Known limits
- Demo passwords are in the code, for testing only.
- Demo accounts are re-created at every start, so a changed role or a deleted demo account is reset on restart.
- The browser session uses an HttpOnly cookie rather than a token in `localStorage`; the Bearer header is accepted for API clients.
- Copilot "this week" questions use the real current date; the generated feedback data may contain no recent reviews, in which case the Copilot says so and shows the latest negative ones.
- Locking after 5 wrong passwords is permanent until an Admin unlocks (no timer).
- `RATE_LIMIT_OFF=1` is set in the launcher for testing.

---
## 17. Submission checklist
- [ ] Notebook outputs cleared (Edit → Clear all outputs) and no key pasted anywhere
- [ ] No secrets, `.db` files or tokens in the repository
- [ ] Folder `Milestone2` in GitHub with the notebook and this README

**Screenshots to add**
- [ ] Login page and sign-up page
- [ ] Profile page
- [ ] Admin dashboard with the users table
- [ ] A denied (403) action for a restricted role
- [ ] The role matrix (in this README)
- [ ] AI Copilot answering an exact question and a greeting
- [ ] A page of each of the six agents

<img width="1600" height="758" alt="aalogin" src="https://github.com/user-attachments/assets/010befe9-326a-4e61-b0a3-0797a46ac8a1" />
<img width="522" height="810" alt="image" src="https://github.com/user-attachments/assets/caf47678-70a2-4bc8-9bfe-fda9658613f2" />
<img width="511" height="518" alt="image" src="https://github.com/user-attachments/assets/913668de-f900-4965-a68f-52ad6c691446" />
<img width="1917" height="898" alt="image" src="https://github.com/user-attachments/assets/b3818fca-dc21-4c89-babb-af878d0cd2ed" />
<img width="1917" height="911" alt="image" src="https://github.com/user-attachments/assets/9c782a59-4374-423d-9489-ec7d5753b3bc" />
<img width="1917" height="896" alt="image" src="https://github.com/user-attachments/assets/d900e761-79d8-475f-b740-2ce591b7dcf0" />
<img width="1713" height="881" alt="image" src="https://github.com/user-attachments/assets/4e4d1898-de6c-435d-b9e7-41e4a7f72d4a" />
<img width="1917" height="921" alt="image" src="https://github.com/user-attachments/assets/0db608c9-7640-42e6-bf43-b768ef37a406" />
<img width="1917" height="906" alt="image" src="https://github.com/user-attachments/assets/e541e758-1a2b-47a6-98a2-cc07616d304c" />
<img width="1917" height="897" alt="image" src="https://github.com/user-attachments/assets/d415c8d8-8f35-4b9f-a130-e5f107fa33ad" />





