🏢 Sairat Sales Management
📋 Overview

Sairat Sales Management is a web-based application designed to streamline and digitize daily sales operations across multiple branches. The system enables administrators to manage branch-wise sales, monitor performance, and track supply and vendor activities — all from a centralized dashboard.

This platform eliminates the need for manual tracking and physical supervision by providing an organized and data-driven approach to handle daily, weekly, and monthly sales across various cities and branches.

🎯 Purpose

The main purpose of Sairat Sales Management is to:

Manage daily sales entries efficiently.

Generate summarized sales reports (daily, weekly, monthly).

Track and monitor multiple branches, their owners, and managers across cities.

Handle items, supply records, and vendor transactions virtually.

Maintain audit logs for better transparency and accountability.

Enable admins to monitor all branch activities from a single platform.

⚙️ Key Features

📊 Sales Management — Record and view daily, weekly, and monthly sales.

🏬 Branch Management — Manage all branches, owners, and city-based divisions.

🧾 Item & Supply Tracking — Maintain item lists, supply entries, and vendor details.

🧑‍💼 User Roles & Access Control — Role-based access for Admins, Managers, and Branch Owners.

🔐 Secure Authentication — JWT-based authentication with httpOnly cookies.

🕵️ Audit Logs — Automatic logging of user and system activities.

📈 Centralized Dashboard — Monitor all operational data in one place.

🖥️ Tech Stack
🧩 Frontend — sairat-admin-web
Technology	Description
ReactJS + Vite	High-performance frontend framework for fast development
TailwindCSS	Utility-first CSS for responsive, modern UI
ShadCN UI	Customizable and accessible UI components
Zustand	Lightweight state management
Axios	For API communication
JWT (httpOnly)	Secure user authentication
⚙️ Backend — sairat-admin-api
Technology	Description
Node.js + Express	Robust backend framework for RESTful APIs
PostgreSQL	Reliable and scalable relational database
Prisma ORM	Modern database ORM for schema management
Zod	Type-safe input validation
JWT	Token-based authentication for secure API access
🧭 Project Setup
🔹 Frontend Setup (sairat-admin-web)
# Clone repository
git clone git@github.com:abp437/sairat-admin-web.git
cd sairat-admin-web

# Copy environment file
cp .env.example .env

# Install dependencies
npm install

# Run development server
npm run dev


✏️ Edit .env to include your backend API base URL and environment-specific configurations.

🔹 Backend Setup (sairat-admin-api)
# Clone repository
git clone git@github.com:abp437/sairat-admin-api.git
cd sairat-admin-api

# Copy environment variables
cp .env.example .env


🎯 Update .env with your PostgreSQL credentials, JWT secret, and environment configs.

# Install dependencies
npm install

# Generate Prisma client
npx prisma generate

# Run migrations
npm run migrate

# Seed initial data (optional)
npx prisma db seed

# Start server
npm run dev

🔄 Schema & Database Management

Whenever you modify Prisma models or add new entities, run:

npx prisma migrate dev --name <migration_name>
npx prisma generate

📦 Modules Overview
Module	Description
Auth	Handles login, logout, and JWT-based authentication
Users	Manage admins, managers, and branch owners
Branches	CRUD operations for branch management
Sales	Daily, weekly, and monthly sales entries
Items & Supplies	Item tracking, vendor supplies, and stock management
Vendors	Manage supplier details and supply entries
Audit Logs	Track all critical activities performed in the system

🧑‍💻 Developed By

Team Sairat Developers

Building a smarter way to manage your sales virtually.

🌐 Future Enhancements

🔄 Automated sales summary generation

📊 Advanced analytics and reports

📱 Mobile app version for managers


🔐 Authentication Module
(Login, Refresh Token, and Role-Based Access)
📘 Overview

The Authentication Module of the Sairat Sales Management System is responsible for validating users, managing secure sessions, and controlling access based on user roles and devices.
It provides a unified login system that supports authentication via Username or Phone Number (PIN) and ensures secure token-based authorization across different client types — including desktop web, mobile web, and mobile app.

⚙️ Features

Login via Username or Phone Number

Role-based Access Control (RBAC) for Admins, Super Admins, Franchise Owners, Vendors, and Entry-Level users

Access Token (JWT) and Refresh Token mechanism

Secure Token Handling using httpOnly cookies (for web) and token storage (for mobile)

Audit Logging for login and logout events

Rate Limiting to prevent brute-force attacks

Device-Specific Session Handling

🔑 Authentication Flow
1️⃣ Login Request
Frontend Implementation

When a user submits the login form:

Detects whether the entered identifier is a username or phone number.

Validates input (e.g., 10-digit Indian phone number format).

Sends a POST request to the backend with the credentials.

axiosInstance.post("/auth/login", {
  identifier: isPhoneNumber ? username : username.toUpperCase(),
  secret: password,
});

Backend Endpoint

POST /auth/login

This route uses:

Zod Validation (loginSchema) to validate request body

Rate Limiter (20 requests / 15 mins)

User Authentication Logic with bcrypt password comparison

2️⃣ Backend Login Process
Step 1 — Identify Login Type
const isPhoneLogin = /^\+?\d{7,15}$/.test(identifier);
const isUsernameLogin = !isPhoneLogin;


Determines whether the user logs in with a phone number or username.

Step 2 — Fetch User
const user = await getLoginUserWithRawQuery(identifier);


Retrieves the user record with all associated details (role, branch, etc.).

Step 3 — Credential Validation

Compares entered password or PIN with hashed value stored in the database:

isValid = await bcrypt.compare(secret, isPhoneLogin ? userPin : password);

Step 4 — Generate Tokens

If credentials are valid, the backend issues:

Access Token (valid for 15 minutes)

Refresh Token (valid for 7 days)

const accessToken = signAccessToken(user);
const refreshToken = signRefreshToken(user);

Step 5 — Send Response Based on Device Type

The backend identifies the client using the x-client-type header.

Client Type	Token Handling	Method
desktop-web / mobile-web	Refresh Token stored in httpOnly cookie	res.cookie("refreshToken", refreshToken, REFRESH_TOKEN_OPTIONS)
mobile-app	Refresh Token returned in response body	responsePayload.refreshToken = refreshToken
3️⃣ Token-Based Role Validation

Each Access Token contains embedded user role information:

{
  userId,
  username,
  phone,
  fullName,
  role,
  isActive,
  branchIds
}


Roles determine which routes a user can access:

Role	Access
SUPER_ADMIN	Full system access (manage branches, users, vendors)
ADMIN	Branch-level management access
FRANCHISE_OWNER	Access to own branch and related sales data
ENTRY_LEVEL	Access to daily sales entry
VENDOR	Access to supply entry and vendor data only

Frontend uses this role data to control navigation and UI visibility.
Backend uses middleware (adminAccessOnly, vendorAccessOnly, etc.) to enforce route-level access.

4️⃣ Token Refresh Flow
Frontend

When the access token expires, the frontend silently requests a new one using:

POST /auth/refresh-token

Backend

The backend checks for a valid refresh token from:

httpOnly cookie (for web clients)

Request body or Authorization header (for mobile clients)

const decoded = verifyRefreshToken(refreshToken);
const user = await getLoginUserWithRawQuery(decoded.id);


If valid:

Generates a new Access Token

Generates a new Refresh Token

Returns them in the response (or sets cookie)

5️⃣ Logout Flow
Frontend

On logout, the frontend clears the stored access token (and refresh cookie for web).

Backend

Endpoint: POST /auth/logout

For web clients → clears refresh token cookie.

For mobile clients → expects frontend to remove stored tokens.

Logs the logout event via auditLogEntry.

🔒 Security Highlights

JWT Expiration:

Access Token: 15 minutes

Refresh Token: 7 days

Cookie Security:

httpOnly, secure, sameSite: strict for refresh tokens in web clients

Audit Trail:
Every login/logout event is recorded in audit logs for accountability.

Rate Limiting:
Prevents excessive login attempts per IP using express-rate-limit.

🧠 Device Flow Summary
Device Type	Storage	Refresh Mechanism	Security Level
Desktop Web	httpOnly cookies	Auto-refresh via cookie	🔒 High
Mobile Web	httpOnly cookies	Auto-refresh via cookie	🔒 High
Mobile App	Local storage / secure storage	Token refresh via API	🔐 Moderate
📁 Related Files & Functions
File	Responsibility
frontend/src/App.jsx	Handles login form, validations, and API calls
backend/controllers/authController.js	Login, Refresh, and Logout logic
backend/utils/jwt.js	Token generation and verification
backend/middlewares/validationMiddleware.js	Input validation using Zod
backend/middlewares/rateLimiter.js	Request throttling for login route
backend/constants.js	Role and environment constants
backend/config.js	Cookie and security configurations
✅ Example
