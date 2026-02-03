Electric Scooter Rental System External Interface Specification  

This document defines the external interface specifications of the electric scooter rental system, applicable to frontend development, backend implementation, test verification, and course demonstration. The system adopts a Client-Server architecture: the client is built with the UniApp framework (supporting multi-platform compilation to H5, Mini Programs, and native apps); the server provides RESTful HTTP APIs; and data persistence uses a MySQL database. All interfaces have a base path of /api/v1/ and run in a local development environment. Third-party services (payment, maps) are implemented via mock interfaces, with paths prefixed by /mock/.  

2 Interface Overview  
2.1 System Architecture Background  
The system consists of three components:  
- Client: A multi-platform application compiled from UniApp, responsible for user interaction and UI rendering;  
- Server: A monolithic backend application handling business logic, authentication/authorization, and data operations;  
- Data Storage Layer: A MySQL database storing core business data including users, scooters, orders, and feedbacks.  

The client communicates with the server via HTTP/HTTPS protocols; the server interacts with the database via JDBC/ORM drivers.  

2.2 Interface Type Classification  
- Frontend–Backend Interfaces: RESTful APIs called by the client to the server—this constitutes the main content of this document;  
- Internal Module Interfaces: Inter-module calls within the server (e.g., order service invoking scooter service), representing code-level contracts not exposed externally;  
- Third-Party System Interfaces: Integration points with external systems; in this project, these are simulated via /mock/ paths while preserving structural compatibility.  

2.3 Design Principles  
- Resource paths use plural nouns (e.g., /scooters);  
- Operation semantics are expressed via HTTP methods (GET for queries, POST for creation, PUT for updates, DELETE for deletion);  
- All sensitive interfaces require an Authorization header (Bearer Token);  
- Responses are uniformly in JSON format: success responses include a data field, error responses include an error field;  
- Interface paths include a version number (/api/v1/) to support future iterations;  
- Permission checks distinguish between user and admin roles.  

3 Detailed Interface Specifications  
3.1 Core Interface List  
No.   Interface Name   Method   Endpoint   Purpose   Requirement ID
API-001   Get Available Scooters List   GET   /scooters   Query rentable scooters and location info   #4, #5

API-002   User Registration   POST   /users   Create a new user account   #1

API-003   User Login   POST   /auth/login   Validate credentials and return authentication token   #2

API-004   Create Booking   POST   /bookings   Submit scooter reservation request   #5, #6

API-005   Query Personal Booking History   GET   /users/{userId}/bookings   Retrieve user’s historical booking list   #7

API-006   Cancel Booking   DELETE   /bookings/{bookingId}   Terminate unstarted bookings   #8

API-007   Submit Fault Feedback   POST   /feedbacks   User reports device anomaly   #12

API-008   Get Revenue Statistics   GET   /admin/revenue   Admin views operational data   #14

API-009   Process Feedback Record   PUT   /feedbacks/{feedbackId}   Admin updates feedback status   #12

API-010   Get Scooter Location   GET   /scooters/{scooterId}/location   Query real-time coordinates of a specific scooter   #9

API-011   Check User Permissions   GET   /auth/check   Verify token validity and user role   #3

3.2 Detailed Interface Descriptions  

API-001 Get Available Scooters List  
Request Parameters:  
- status (string, optional)  
- page (integer, optional)  
- limit (integer, optional)  

Response Content:  
Scooter list (including ID, coordinates, battery level, status), pagination info  

Status Codes:  
- 200 (success)  
- 400 (parameter error)  
- 500 (server error)  

API-002 User Registration  
Request Parameters:  
- email (string, required)  
- password (string, required)  
- name (string, required)  

Response Content:  
New user ID, email, name  

Status Codes:  
- 201 (created successfully)  
- 400 (missing fields)  
- 409 (email already exists)  

API-003 User Login  
Request Parameters:  
- email (string, required)  
- password (string, required)  

Response Content:  
Authentication token, user ID, role  

Status Codes:  
- 200 (success)  
- 401 (invalid credentials)  

API-004 Create Booking  
Request Parameters:  
- scooter_id (string, required)  
- user_id (string, required)  

Response Content:  
Booking ID, associated scooter ID, user ID, status, creation time  

Status Codes:  
- 201 (success)  
- 401 (unauthorized)  
- 409 (scooter already booked)  

API-005 Query Personal Booking History  
Request Parameters: none (user ID embedded in path)  

Response Content:  
Booking list (including ID, scooter ID, status, start/end times)  

Status Codes:  
- 200 (success)  
- 401 (unauthorized)  
- 404 (user not found)  

API-006 Cancel Booking  
Request Parameters: none (booking ID embedded in path)  

Response Content:  
Booking ID, updated status  

Status Codes:  
- 200 (success)  
- 403 (insufficient permissions)  
- 404 (booking not found)  

API-007 Submit Fault Feedback  
Request Parameters:  
- scooter_id (string, required)  
- description (string, required)  
- location (string, optional)  

Response Content:  
Feedback ID, scooter ID, status, creation time  

Status Codes:  
- 201 (success)  
- 401 (unauthorized)  

API-008 Get Revenue Statistics  
Request Parameters:  
- start_date (date, optional)  
- end_date (date, optional)  

Response Content:  
Total revenue amount, total order count, average order amount  

Status Codes:  
- 200 (success)  
- 403 (insufficient permissions)  

API-009 Process Feedback Record  
Request Parameters:  
- status (string, required)  
- note (string, optional)  

Response Content:  
Feedback ID, updated status, processing note, update time  

Status Codes:  
- 200 (success)  
- 403 (insufficient permissions)  
- 404 (record not found)  

API-010 Get Scooter Location  
Request Parameters: none (scooter ID embedded in path)  

Response Content:  
Latitude, longitude, timestamp  

Status Codes:  
- 200 (success)  
- 404 (scooter not found)  

API-011 Check User Permissions  
Request Parameters: none  

Response Content:  
User ID, role, token expiration time  

Status Codes:  
- 200 (valid)  
- 401 (invalid or expired)  

4 Interface Testing Requirements  
Testing must cover functional correctness, exception handling, response performance, and multi-platform compatibility.  

Functional Testing: Verify each interface returns expected results per acceptance criteria;  
Exception Testing: Test scenarios such as missing parameters, unauthorized access, resource conflicts, etc.;  
Performance Requirement: P95 response time ≤ 800 ms in local environment;  
Compatibility: Ensure UniApp-compiled outputs for H5, WeChat Mini Program, and Android all invoke interfaces correctly;  
Acceptance Criteria: All Must-have interfaces must be covered by automated test cases; demo can be verified on-site.  

5 Appendices  
5.1 HTTP Status Code Reference  
Status Code   Meaning   Usage Scenario
200   Request succeeded   Resource query, status update successful

201   Resource created successfully   User registration, booking creation, etc.

400   Bad request (parameter error)   Missing fields, invalid format

401   Unauthorized   Missing or invalid Token

403   Forbidden   Insufficient permissions (e.g., regular user accessing admin interface)

404   Resource not found   Accessing non-existent order or scooter

409   Conflict   Duplicate operation on same resource (e.g., duplicate booking)

500   Internal server error   Database exception, service crash
