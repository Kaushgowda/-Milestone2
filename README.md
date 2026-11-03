# FranchiseOps AI Copilot – Milestone 2

## 1. Project Overview

**FranchiseOps AI Copilot** is an AI-powered Franchise Management System developed as part of the **Infosys Springboard Internship 7.0 – Batch 4.0**.

The system helps franchise businesses monitor operations, analyze performance, access business information, and make data-driven decisions through an interactive dashboard and an AI-powered SQL Copilot.

Milestone 2 focuses on extending the existing Milestone 1 application by integrating authentication, improving navigation, enhancing the Copilot experience, and preparing the system for role-based access.

## 2. Objectives

- Integrate the authentication system developed in Milestone 1.
- Provide secure login and user registration.
- Build an interactive and responsive franchise management dashboard.
- Provide navigation between different application modules.
- Implement an AI Copilot that understands business-related questions.
- Retrieve answers from the franchise database using SQL queries.
- Display generated SQL queries and query results.
- Support role-based access management as a separate project responsibility.
- Maintain existing application functionality while introducing new features.

## 3. Technology Stack

| Category | Technologies |
|---|---|
| Programming Language | Python |
| Backend Framework | FastAPI |
| Frontend | HTML, CSS, JavaScript |
| Database | SQLite |
| Authentication | JWT, bcrypt |
| Data Processing | Pandas, NumPy |
| Machine Learning | Scikit-learn, SciPy |
| Sentiment Analysis | VADER |
| Data Visualization | Matplotlib, Apache ECharts |
| Maps | Leaflet, TopoJSON |
| PDF Processing | pdfplumber |
| HTTP Requests | Requests |
| Email Service | SMTP |
| Application Server | Uvicorn |
| Development Environment | Google Colab |
| Deployment and Tunnelling | Google Colab Proxy, pyngrok |
| Version Control | Git, GitHub |

## 4. Features and Modules

### 4.1 Authentication

The authentication module extends the login functionality developed in Milestone 1.

**Features include:**

- User login using email and password.
- New user registration.
- Password hashing using bcrypt.
- Password validation.
- Show/hide password option in the login interface.
- Forgot password functionality using a security question.
- Password reset.
- JWT-based authentication sessions.

### 4.2 Dashboard

The dashboard provides a centralized interface for accessing franchise operations information.

**It includes:**

- Franchise operations overview.
- Performance monitoring.
- Interactive dashboard components.
- Navigation between application modules.
- Copilot access.
- Workforce-related information.

### 4.3 AI Copilot

The AI Copilot accepts natural-language questions and retrieves relevant information from the franchise database using SQL queries.

It supports business-related questions involving outlets, workforce, inventory, sales, and other available database information.

**Key features include:**

- Understands different wordings of supported questions.
- Handles common spelling variations.
- Retrieves answers from the SQLite database.
- Displays the SQL query used to generate the answer.
- Displays query results and row counts where applicable.
- Provides helpful responses for greetings and unsupported questions.
- Applies outlet-level data restrictions where required by the user's role.

### 4.4 Navigation and Routing

The application uses FastAPI routes to navigate between the login interface, dashboard, and backend API endpoints.

| Route | Purpose |
|---|---|
| `/login` | Displays the login page |
| `/` | Main application dashboard |
| `/api/health` | Checks server availability |
| `/api/auth/signup` | User registration |
| `/api/auth/login` | User login |
| `/api/auth/me` | Retrieves authenticated user information |
| `/api/auth/logout` | Logs out the user |
| `/api/auth/forgot` | Password recovery |
| `/api/page/copilot` | Copilot page data |
| `/api/page/workforce` | Workforce page data |
| `/api/chats` | Chat-related data |
| `/api/stream` | Application streaming endpoint |

*Note: API routes are backend endpoints, whereas `/` and `/login` are browser-facing pages.*

### 4.5 Role-Based Access Management

The overall project requirements identify five user roles:

1. Admin
2. Franchise Owner
3. Store Manager
4. Staff
5. Auditor

Role-based permissions for pages, database access, and actions are being handled separately by the designated team member.

This README does not claim that RBAC has been completed as part of the authentication integration.

## 5. Copilot – Sample Questions and Answers

The following examples demonstrate the expected question-and-answer format of the Copilot.

| No. | Sample Question | Expected Answer |
|---|---|---|
| 1 | Which is the healthiest outlet? | Displays the outlet with the highest health score, along with its location and score. |
| 2 | Which outlet has the highest risk? | Displays the outlet with the lowest health score and its weakest performance area. |
| 3 | Why is OUT-010 flagged? | Explains the outlet's health status, weakest agent scores, and open action tickets, if available. |
| 4 | What is the current inventory status? | Displays the number of SKU records, items at or below reorder level, items with less than seven days of stock cover, and stock-out risk. |
| 5 | Which items need urgent restocking? | Displays the most urgent SKUs, quantities to order, and outlets requiring stock replenishment. |
| 6 | Show the sales trend for the last seven days. | Displays recorded sales for the last seven days, including the best and weakest sales days. |
| 7 | How many employees work in Bengaluru? | Displays the employee count for Bengaluru and the location with the highest employee count, where available. |
| 8 | Which city has the most employees? | Displays the city with the highest employee count and the employee count for each listed city. |
| 9 | Which state has the most outlets? | Displays the state with the highest number of outlets. |
| 10 | Show all outlets in Karnataka. | Displays the outlets matching the selected state, where supported by the city-to-state mapping. |
| 11 | What is the workforce status? | Displays total staff, employees with high attrition risk, and average overtime hours. |
| 12 | Show details of OUT-010. | Retrieves matching outlet records and related available action-ticket information. |

### 5.1 Additional Example Business Questions

Other examples of natural-language questions include:

- Show me the overall franchise performance.
- What is the current workforce status?
- Show the available franchise information.
- Give me the relevant business performance details.
- Retrieve the records matching my question.
- Show the database results for this query.

### 5.2 Copilot Response Details

When a supported question is submitted:

1. The Copilot identifies the question pattern.
2. It retrieves the relevant information from the database.
3. It executes the corresponding SQL query.
4. It displays the generated SQL query.
5. It presents the query results and row count, where applicable.

Unsupported questions receive a helpful response with suggestions for supported queries.

**Note:** Answers are generated dynamically from the database. The examples above describe expected response content, not fixed database values. Actual results depend on the available records, implemented SQL patterns, and user permissions. The exact questions supported depend on the implementation in `analyst.py` and `qa.py`.

## 6. Application Screenshots

### 6.1 Login Page

<img width="1600" height="758" alt="aalogin" src="https://github.com/user-attachments/assets/010befe9-326a-4e61-b0a3-0797a46ac8a1" />

### 6.2 Main Dashboard

![Main Dashboard](Screenshots/Dashboard.png)

### 6.3 AI Copilot Interface

![AI Copilot](Screenshots/Copilot.png)

### 6.4 Copilot SQL Query and Results

![Copilot SQL Results](Screenshots/Copilot_query.png)

### 6.5 Navigation and Application Modules

![Application Navigation](Screenshots/Navigation.png)

## 7. Project Structure

The intended Milestone 2 repository structure is:

```text
Milestone2/
│
├── README.md
├── FranchiseOps_Copilot_Login_Integrated.ipynb
│
└── Screenshots/
    ├── Login.png
    ├── Dashboard.png
    ├── Copilot.png
    ├── Copilot_query.png
    └── Navigation.png
```

The application source code generated and executed in the notebook is organized as follows:

```text
app/
│
├── server.py
├── auth.py
├── db.py
├── config.py
├── analyst.py
├── qa.py
├── live_engine.py
├── worker.py
│
└── static/
    ├── index.html
    ├── login.html
    ├── app.js
    ├── login.js
    ├── style.css
    └── vendor/
```

## 8. Installation and Execution

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### Step 2: Install Dependencies

```bash
pip install fastapi uvicorn bcrypt PyJWT email-validator pandas
```

Install any additional dependencies required by the application.

### Step 3: Prepare the Application

Ensure that the application files, static assets, and database are available in the expected directories.

### Step 4: Start the FastAPI Server

```bash
cd app
uvicorn server:app --host 0.0.0.0 --port 8000
```

### Step 5: Open the Application

Open the following address in your browser:

```text
http://localhost:8000/login
```

After successful authentication, the user can access the main application.

*For Google Colab execution, use the notebook's server-startup and Colab proxy-link cells instead of opening localhost directly.*

## 9. Testing

The existing Copilot functionality was tested for:

- Supported SQL question patterns.
- Question wording and spelling normalization.
- SQL query execution.
- Query result display.
- Row-count display.
- Greeting detection.
- Unsupported question handling.
- Suggestions for unsupported questions.

The authentication module includes login, registration, and password recovery endpoints. Complete end-to-end authentication testing should be performed before considering the integration production-ready.

## 10. Milestone 2 Deliverables

- Integration of the Milestone 1 authentication functionality.
- Login and registration interface.
- Password hashing and JWT authentication.
- Password recovery functionality.
- Existing FranchiseOps dashboard retained.
- AI SQL Copilot functionality retained.
- Application navigation and routing.
- Project documentation.
- Application screenshots.
- Preparation for separate RBAC integration.

## 11. Team Members and Contributions

| Team Member | Contribution |
|---|---|
| Kaushik B  | Role-Based Access Control (RBAC) Integration |
| Moghitha B | Login and Authentication Integration, AI Copilot Enhancement |
| Pullaiahgari Lahari | Streamlit to FastAPI Integration |

## 12. Conclusion

Milestone 2 extends the FranchiseOps AI Copilot by integrating authentication into the existing franchise management application while retaining its dashboard and database-driven Copilot capabilities.

The application aims to provide a centralized platform where franchise users can access business information, interact with an AI-powered Copilot, and monitor operational performance through a structured interface.
**Screenshots**
<img width="1600" height="758" alt="aalogin" src="https://github.com/user-attachments/assets/010befe9-326a-4e61-b0a3-0797a46ac8a1" />
<img width="511" height="518" alt="image" src="https://github.com/user-attachments/assets/913668de-f900-4965-a68f-52ad6c691446" />
<img width="1917" height="898" alt="image" src="https://github.com/user-attachments/assets/b3818fca-dc21-4c89-babb-af878d0cd2ed" />
<img width="1917" height="911" alt="image" src="https://github.com/user-attachments/assets/9c782a59-4374-423d-9489-ec7d5753b3bc" />
<img width="1917" height="896" alt="image" src="https://github.com/user-attachments/assets/d900e761-79d8-475f-b740-2ce591b7dcf0" />
<img width="1713" height="881" alt="image" src="https://github.com/user-attachments/assets/4e4d1898-de6c-435d-b9e7-41e4a7f72d4a" />
<img width="1917" height="921" alt="image" src="https://github.com/user-attachments/assets/0db608c9-7640-42e6-bf43-b768ef37a406" />
<img width="1917" height="906" alt="image" src="https://github.com/user-attachments/assets/e541e758-1a2b-47a6-98a2-cc07616d304c" />
<img width="1917" height="897" alt="image" src="https://github.com/user-attachments/assets/d415c8d8-8f35-4b9f-a130-e5f107fa33ad" />





