# FranchiseOps AI — Milestone 2 (Infosys Springboard 7.0, Batch 4)

FastAPI + SQLite + JWT + RBAC, run in Google Colab (ngrok as fallback).

## How to run
1. Open `Milestone2_FranchiseOps_Copilot_Final.ipynb` in Google Colab → **Runtime → Run all**.
2. Add Colab Secrets (key icon): `JWT_SECRET`, `EMAIL_ADDRESS`, `EMAIL_PASSWORD` (Gmail app password, for OTP mails), `NGROK_AUTHTOKEN` (fallback link). Never type a secret in a cell.
3. Wait for `FranchiseOps AI is running`, then click **Open the app**.
4. Optional: run the last cell (role self-test) to see allowed / denied (403) results for every role.

## Login (Step 3)
Sign up, sign in, forgot password (security question or e-mailed OTP): `POST /api/register`, `/api/login`, `/api/login/admin`, `/api/reset/*`, `/api/otp/*`, `/api/logout`. Users are in the SQLite `users` table with bcrypt password hashes; the JWT (signed with `JWT_SECRET`, 2 h) travels in an HttpOnly cookie, or in `Authorization: Bearer`. `user_of()` re-reads the user from the database on **every** request, so a locked or deleted user is blocked immediately. Unknown e-mail and wrong password return the same text; 5 wrong passwords lock the account until an Admin unlocks it. Validation rules are the Milestone 1 rules.

## Roles and privileges (Step 5) — enforced on the server (`rbac.py`)
| Role | Copilot | Workf. | Outlets | Invent. | Mktg | Sentim. | Audit | Run agents | Users | Outlet scope |
|---|---|---|---|---|---|---|---|---|---|---|
| Admin | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | all |
| Franchise Owner | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | all |
| Store Manager | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ | own outlet |
| Staff | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | own outlet |
| Auditor | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | all (read-only) |

`ROLE_MENU` and `ROLE_CAPS` live in `rbac.py`. `menu_for(user)` returns only allowed pages; `/api/page/<id>` returns **403** for any other page; `need(ctx, cap)` raises 403 in every action route. Store Manager and Staff are tied to one outlet (`users.outlet`; `ctx["scope"]`): every page, detail drawer, pulse counter and Copilot query is filtered by it. Auditor is read-only (no run, no inspection logging, no reviews).

Demo accounts (testing only): `admin@infosys.com` / `Admin@123` (Administrator Login, username `admin`), `owner@infosys.com`, `manager@infosys.com`, `staff@infosys.com`, `auditor@infosys.com` (password `Demo@1234`). Manager and Staff work at the first outlet. The Colab secrets `ADMIN_EMAIL_ID` / `ADMIN_PASSWORD` override the admin account.

## Profile (Step 4) and Admin dashboard (Step 6)
Profile: username, e-mail, role, outlet, joined, last login; change username; change password (current password first, Milestone 1 rules). The hash is never shown.
Admin dashboard (`need(ctx,"users")` on every route): KPI cards (total, per role, locked), users table, add user, promote/demote, set outlet, lock/unlock, delete with a two-step confirmation, role permission table, activity log (latest 50). Rules: you cannot delete, lock or change your own account; the last remaining Admin can never be deleted, locked or demoted; the built-in Administrator Login account is protected.

## Copilot (Step 7)
Every answer is one read-only SQL query; the SQL and the number of rows are shown. Spelling mistakes are fixed (`negitive reveiws`, `lowst satisfiction`), synonyms are understood (`shops`, `employees`, `CSAT`, `running low`). Outside the data it replies politely with three valid questions. RBAC: a role can only read topics it may see (Store Manager: no marketing; Staff: outlets, inventory, feedback only) and only its own outlet.

28 new intents (`analyst.py`): lowest/highest CSAT · staff count of an outlet · highest attrition outlets · employees at high attrition risk · average attrition · most overtime · items below reorder level · highest stock-out risk · lowest stock cover · best/lowest ROI campaign · best ROI channel · most conversions · marketing budget by channel · negative reviews this week · outlet with most negative feedback · latest negative reviews · average rating by outlet · lowest-rated reviews · action-required audits · pending compliance · lowest audit scores · top/lowest revenue outlets · most profitable outlets · open tickets by severity · number of outlets.

Tested questions (Franchise Owner; all answered from SQL):
| Question | Answer (from the database) |
|---|---|
| which outlet has the lowest CSAT | OUT-110 (2.95) |
| how many staff does OUT-010 have | 14 |
| list items below reorder level | 12 items (OUT-030 Frozen Fries 32/79 first) |
| which campaign has the best ROI | Campaign 024, OUT-041, 5.37x |
| show negative reviews this week | 20 negative reviews (last 7 days) |
| show negitive reveiws this week (typo) | same as above |
| whcih outlet has the lowst satisfiction (typos) | OUT-110 (2.95) |
| how many outlets are there | 120 |
| how many open tickets | by severity from `agent_actions` |
| which outlet has the lowest stock cover | OUT-094 (7.83 days) |
| tell me a joke | polite refusal + 3 valid questions |
Values depend on the generated data and change between runs. As Store Manager the same lowest-CSAT question returns only your own outlet; "which campaign has the best ROI" is refused; "how many staff does OUT-005 have" says it is outside your scope.

## Design (Step 8)
Consistent colour theme, light/dark switch (remembered), readable zebra tables, responsive layout for phones, loading spinners, "Querying the database…" in the Copilot, clear error messages and a 403 "Access denied" page.

## Submission checklist
Clear outputs (this notebook is saved without outputs), no secrets, no `.db` files in the repository. Screenshots to add: login/sign-up, profile, admin dashboard, a 403, the role matrix, Copilot greeting + exact answer, one page per agent.
