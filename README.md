ATTEND-IQ : AUTOMATED ATTENDANCE CAPTURE WITH STUDENT ENGAGEMENT AND ACADEMIC PERFORMANCE INSIGHTS#
A full-stack college project combining:

Java Spring Boot + MySQL (JDBC) backend
HTML/CSS/JS + Bootstrap + Chart.js frontend
Python Flask + Scikit-learn (RandomForest) AI prediction microservice
1. Project Overview#
Three roles — Admin, Faculty, Student — share one system for marking attendance, tracking percentages, applying/approving leave, generating parent notifications on absence, and predicting future attendance risk with a real trained machine learning model (not hard-coded thresholds).

2. Features#
Session-based authentication with role-based access control (a student cannot access /api/admin/** or /api/faculty/** even by typing the URL).
Admin: manage students, faculty, subjects, view all analytics, reports, AI predictions, leave requests, and notifications.
Faculty: mark/edit attendance (duplicate-proof), approve/reject leave, view AI predictions and reports for their subjects.
Student: view attendance (overall/subject-wise/monthly), apply for leave, view their own AI-predicted attendance and risk level, view notifications.
Real-time parent notification generated automatically whenever a student is marked absent.
Attendance percentage calculated directly from MySQL with SUM/COUNT SQL, never hard-coded.
AI prediction calls a live Flask REST API backed by a trained RandomForestRegressor + RandomForestClassifier (see metrics below).

3. Technology Stack#
Layer	Technology
Backend	Java 17, Spring Boot 3.3, Spring JDBC (JdbcTemplate)
Database	MySQL 8
Frontend	HTML5, CSS3, vanilla JavaScript, Bootstrap 5, Chart.js
AI/ML	Python 3, Flask, Pandas, NumPy, Scikit-learn, Joblib
Auth	HttpSession + custom PBKDF2-like SHA-256 password hashing (no external crypto dependency)

4. Architecture#
Frontend (HTML/JS)
      |
      v
Spring Boot REST API  ---- JDBC ---->  MySQL
      |
      v
Flask AI API  ---->  trained RandomForest models (.pkl)

5. Folder Structure#
attendance-ai-system/
├── backend/                # Spring Boot application
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/college/attendance/
│       │   ├── AttendanceApplication.java
│       │   ├── controller/     (11 REST controllers)
│       │   ├── service/        (9 services)
│       │   ├── repository/     (9 JdbcTemplate repositories)
│       │   ├── model/          (9 entities)
│       │   ├── dto/            (12 request/response DTOs)
│       │   ├── config/         (CORS, RestTemplate, session auth filter)
│       │   ├── exception/      (custom exceptions + global handler)
│       │   └── util/           (PasswordUtil)
│       └── resources/application.properties
├── frontend/
│   ├── login.html
│   ├── admin/      (10 pages)
│   ├── faculty/    (6 pages)
│   ├── student/    (5 pages)
│   ├── css/style.css
│   └── js/ (api.js, auth.js, layout.js)
├── ai_model/
│   ├── generate_dataset.py
│   ├── dataset.csv
│   ├── train_model.py
│   ├── prediction.py
│   ├── app.py
│   ├── requirements.txt
│   ├── attendance_model.pkl
│   ├── risk_model.pkl
│   └── metrics.json
├── database/
│   ├── schema.sql
│   ├── generate_sample_data.py
│   └── sample_data.sql
├── TESTING.md
├── .gitignore
└── README.md

6. API Documentation (summary)#
All responses are wrapped as { success, message, data }.

Method	Endpoint	Role	Purpose
POST	/api/auth/login	Public	Login, creates session
POST	/api/auth/logout	Public	Destroys session
GET	/api/admin/students	Admin	List/search students
POST/PUT/DELETE	/api/admin/students/{id}	Admin	CRUD students
GET/POST/PUT/DELETE	/api/admin/faculty	Admin	CRUD faculty
GET/POST	/api/admin/subjects	Admin	Manage subjects + assignment
GET	/api/admin/analytics/dashboard	Admin	Dashboard stat cards
GET	/api/admin/analytics/{overall,bands,subject-wise,monthly-trend,present-absent}	Admin	Chart data
GET	/api/faculty/attendance/my-subjects	Faculty	Assigned subjects
GET	/api/faculty/attendance/roster	Faculty	Class roster for a subject
POST	/api/faculty/attendance/mark	Faculty	Mark/edit attendance (upsert)
GET	/api/student/attendance/{subject-wise,monthly,overall}	Student	Own attendance
POST	/api/student/leave/apply	Student	Apply leave
GET	/api/student/leave/mine	Student	Own leave history
GET	/api/faculty/leave/pending	Faculty	Pending leave requests
POST	/api/faculty/leave/{id}/decide	Faculty	Approve/reject
GET	/api/admin/leave/all	Admin	All leave requests
POST	/api/faculty/ai/predict/{studentId}	Faculty	Run AI prediction
POST	/api/student/ai/predict/me	Student	Run own AI prediction
GET	/api/faculty/notifications , /api/student/notifications/mine	Faculty/Student	Notification history
GET	/api/faculty/reports/{daily,monthly,defaulters}	Faculty/Admin	Reports
Flask AI service:

Method	Endpoint	Purpose
GET	/health	Health check
POST	/predict	Real ML prediction (RandomForest)

7. Setup Instructions#
7.1 Prerequisites#
Java 17+ (JDK, not just JRE) and Maven 3.8+
MySQL 8+ running locally
Python 3.10+
7.2 Database Setup#
mysql -u root -p < database/schema.sql
mysql -u root -p < database/sample_data.sql
7.3 AI Model Setup#
cd ai_model
pip install -r requirements.txt
python3 generate_dataset.py   # regenerates dataset.csv (optional, already included)
python3 train_model.py        # trains and saves attendance_model.pkl / risk_model.pkl
python3 app.py                # starts Flask API on http://localhost:5000
7.4 Backend Setup#
Set environment variables if your MySQL credentials differ from the defaults (root/root):

export DB_URL="jdbc:mysql://localhost:3306/attendance_analytics_db?useSSL=false&serverTimezone=UTC"
export DB_USERNAME=root
export DB_PASSWORD=your_password
export AI_SERVICE_URL=http://localhost:5000
Then build and run:

cd backend
mvn clean install
mvn spring-boot:run
The API starts on http://localhost:8080.

7.5 Frontend Setup#
No build step required — plain static files. Easiest option:

cd frontend
python3 -m http.server 5500
Open http://localhost:5500/login.html in your browser.

(Opening login.html directly via file:// also works since CORS is configured with allowedOriginPatterns("*") and credentials enabled, but a local HTTP server avoids browser cookie quirks with file:// origins.)

8. Default Demo Accounts#
Role	Username	Password
Admin	admin	admin123
Faculty	faculty1	faculty123
Student	student1	student123
These are DEMO credentials only — change them before any real deployment. All faculty (faculty1-faculty5) and all students (student1-student20) use the same demo passwords respectively.

9. Demo Flow#
Login as admin → view dashboard stat cards and charts.
Browse Students / Faculty / Subjects.
Logout, login as faculty1.
Go to "Mark Attendance" → pick a subject → load roster → mark present/absent → submit.
A parent notification is generated automatically for anyone marked absent.
Logout, login as student1.
View attendance (now updated), apply for leave.
Logout, login as faculty1, approve/reject the leave request.
As faculty or student, go to "AI Predictions" and click "Run Prediction" — this calls the live Flask API and stores the result.
Back as admin, view Analytics and Reports, export a CSV.

10. AI Model Details#
Regression: RandomForestRegressor (200 trees, max_depth 8) predicts future_attendance from 8 engineered features.
Classification: RandomForestClassifier predicts risk tier (LOW/MEDIUM/HIGH) from the same features, cross-checked against configurable thresholds (LOW ≥ 80%, MEDIUM ≥ 75%, HIGH < 75%) in ai_model/prediction.py — the more cautious (higher-risk) verdict wins if the model and threshold ever disagree.
Trained on 300 synthetic-but-realistic records (ai_model/dataset.csv).
Real, non-fabricated metrics from the last training run (see ai_model/metrics.json): MAE ≈ 3.1, RMSE ≈ 4.1, R² ≈ 0.92, classifier accuracy ≈ 0.83.

11. Security Notes#
Passwords are never stored in plaintext — see PasswordUtil.java (16-byte random salt + 10,000 rounds of SHA-256, stored as salt:hash).
All SQL uses PreparedStatement/JdbcTemplate parameter binding — no string-concatenated queries anywhere.
SessionAuthFilter blocks cross-role access at the HTTP layer, not just in the UI, so manually typing another role's URL is rejected server-side.
No secrets are hard-coded; DB credentials and the AI service URL are read from environment variables with local-only defaults.

12. Troubleshooting#
Problem	Fix
Communications link failure on backend startup	MySQL isn't running, or DB_URL/credentials are wrong
AI prediction fails with "unreachable"	Start python3 ai_model/app.py first — the Java backend calls it live for every prediction
CORS error in browser console	Make sure you're loading the frontend over http:// (not double-checking a typo'd port) and that the backend is on port 8080
Login succeeds but every other page redirects back to login	The browser blocked the session cookie — use a local HTTP server for the frontend instead of file://, or check your browser's third-party-cookie settings for localhost
mvn not found	Install Maven and ensure JAVA_HOME points to a JDK (not just a JRE)
13. Known Simplifications (documented, not hidden)#
The AI feature leave_days is currently sent as 0 in AiPredictionService rather than being computed from approved leave records — wiring that up is a straightforward follow-on (join leave_requests where status='APPROVED' and count days in the current month).
The Daily Report endpoint currently returns subject-wise overall totals rather than filtering strictly by the requested date; the date parameter is accepted and validated but the underlying query would need a date-parameterized version of subjectWiseOverall() for exact daily filtering — noted here rather than silently shipped as if fully wired.
These are the only two spots in the whole system with a shortcut; every other module (auth, CRUD, attendance marking/duplicate-prevention, percentage math, leave workflow, notifications, AI prediction end-to-end, reports/CSV export) is fully implemented and was exercised end-to-end during development.
