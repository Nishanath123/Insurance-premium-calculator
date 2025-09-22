# Insurance-premium-calculator README
Microservice-based insurance premium calculator


## Overview
The Insurance Premium Calculator is a microservice-oriented REST API system designed to calculate insurance premiums for customers based on various factors such as policy details, customer risk profiles, add-on coverages, and regional multipliers. This project modernizes a traditional monolithic insurance backend into a distributed microservices architecture using Django (Python web framework). Each service is independent, communicates via REST APIs, and uses JWT (JSON Web Tokens) for authentication on protected endpoints (primarily POST operations).

The system consists of six microservices:
1. **Auth Service**: Handles user authentication and JWT token generation.
2. **Policy Service**: Manages insurance policies and their base premiums.
3. **Customer Service**: Manages customer profiles and risk factors.
4. **Coverage Service**: Manages add-on coverage options and customer selections.
5. **Geo Service**: Manages region-based premium multipliers.
6. **Premium Service**: Acts as an aggregator to calculate the final premium by orchestrating calls to the other services.

### Key Features
- **Microservices Architecture**: Each service runs independently, allowing for scalability, easier maintenance, and fault isolation.
- **Authentication**: JWT-based authentication secures POST endpoints. GET endpoints are public for simplicity.
- **Database**: Each service uses PostgresSQL (via Django ORM) for data persistence. 
- **Validation**: Manual input validation without Django serializers, throwing specific error responses (e.g., `{"name": ["This field is required."]}`) if required fields are missing.
- **Premium Calculation Logic** (assumed and implemented in Premium Service):
  - Age Factor: 1.0 (age < 30), 1.2 (30 ≤ age < 50), 1.5 (age ≥ 50).
  - Risk Factor: 1.0 base + 0.1 (if smoker) + 0.2 (if has illness).
  - Add-ons: Sum of selected coverage costs.
  - Final Premium: (Base Premium * Age Factor * Risk Factor * Region Multiplier) + Add-ons.
- **Inter-Service Communication**: Premium Service makes HTTP requests to other services to fetch data.
- **No External Dependencies Beyond Basics**: Requires Django, Django REST Framework, PyJWT, and Requests. No serializers used; ORM handles database interactions.

This project demonstrates RESTful API design, authentication integration, and microservice orchestration. It's suitable for educational purposes or as a starting point for a real insurance backend.

## Prerequisites
- Python 3.8+ installed.
- Virtual environment recommended (e.g., `python -m venv venv; source venv/bin/activate`).
- Install dependencies in each service directory: `pip install django djangorestframework pyjwt requests`.
- Each service uses a shared JWT secret ('secret' hardcoded; use environment variables in production).

## Setup Instructions
1. **Clone/Create the Project Structure**:
   - Create a root directory (e.g., `Insurance Premium Calculator`).
   - Inside it, create subdirectories for each service: `Auth_Service`, `Policy_Service`, `Customer_Service`, `Coverage_Service`, `Geo_Service`, `Premium_Service`.
   - In each subdirectory:
     - Run `django-admin startproject <project_name> .` (e.g., `auth_service` for Auth_Service).
     - Navigate into the project folder (e.g., `cd auth_service`).
     - Run `python manage.py startapp <app_name>` (e.g., `auth_app` for Auth_Service, `policy_app` for Policy_Service, etc.).
   - Note: The code examples below assume app names like `auth_app`, `policy_app`, etc. Adjust if using 'api' as in earlier code.

2. **Configure settings.py** (Common for All Services):
   - Replace the content of `<service_name>/settings.py` with the provided template (from the code snippet). Update `ROOT_URLCONF` and `WSGI_APPLICATION` to match your project name (e.g., 'auth_service.urls').
   - Add `'rest_framework'` and our app (e.g., `'auth_app'`) to `INSTALLED_APPS`.
   - Ensure `DATABASES` uses PostgresSQL.

3. **Apply Code to Each Service**:
   - Copy models.py, views.py, and urls.py from the provided code snippets into the respective app directories (e.g., `auth_app/models.py`).
   - Note: In the code, app was 'api', so rename imports/references to match your app name (e.g., change `from api.models` to `from auth_app.models`).
   - For Auth Service: Run `python manage.py createsuperuser` to create an admin user (e.g., username: admin, password: pass).

4. **Run Migrations**:
   - In each service directory: `python manage.py makemigrations <app_name>; python manage.py migrate`.

5. **Run the Services**:
   - Run each on different ports (use separate terminals):
     - Auth: `python manage.py runserver 8000`
     - Policy: `python manage.py runserver 8001`
     - Customer: `python manage.py runserver 8002`
     - Coverage: `python manage.py runserver 8003`
     - Geo: `python manage.py runserver 8004`
     - Premium: `python manage.py runserver 8005`
   - Services communicate via localhost URLs (hardcoded in Premium Service; configure via env vars in production).

6. **Optional: Docker Setup**:
   - Create a `Dockerfile` in each service directory:
     ```
     FROM python:3.9-slim
     WORKDIR /app
     COPY . /app
     RUN pip install -r requirements.txt  # Create requirements.txt with dependencies
     CMD ["python", "manage.py", "runserver", "0.0.0.0:<port>"]
     ```
   - Use docker-compose.yml in root for orchestration (not provided here).

7. **Testing**: Use Postman or curl for API calls. First, obtain JWT from Auth Service.

## Modules Present and Their Usage
Each service is a Django project with one app. Modules (files) include:
- **settings.py**: Configures Django settings, apps, middleware, databases, and REST Framework parsers.
- **urls.py**: Defines URL patterns routing to views.
- **models.py** (in app): Defines ORM models for database tables.
- **views.py** (in app): Contains API view functions for endpoints, including JWT validation, input checks, and business logic.
- **manage.py**: Django's command-line utility for migrations, running server, etc.

### Functionality of Every Code (Descriptive)
- **settings.py** (All Services): This is the core configuration file. It sets up the project structure, enables debugging, defines installed apps (including REST Framework for API handling), configures middleware for security and sessions, sets up SQLite database, and defines REST parsers for JSON input. For Auth Service, it integrates Django's auth system. Functionality: Ensures the service runs with proper environment; without it, Django won't start.
  
- **urls.py** (All Services): Routes incoming HTTP requests to appropriate views. Includes admin paths and API endpoints. Functionality: Acts as the entry point for API calls; e.g., maps `/policies/` to `add_policy` view.

- **models.py** (In Apps, except Premium Service): Uses Django ORM to define database schemas. E.g., Policy model has fields like `name` (CharField) and `base_premium` (DecimalField). Functionality: Handles CRUD operations via ORM queries (e.g., `Policy.objects.create()`); abstracts SQL for portability.

- **views.py** (In Apps): Core logic for APIs.
  - **validate_jwt()**: Helper function to check JWT in headers; decodes with shared secret. Functionality: Secures endpoints; returns 401 if invalid.
  - **POST Views** (e.g., add_policy): Validates required fields manually (no serializers), creates ORM objects, returns created data or errors. Functionality: Enforces data integrity; e.g., checks if `name` is None and responds with specific error JSON.
  - **GET The user is asking for a README documentation for the Insurance Premium Calculator project. It should include the use of it, modules present, use of the modules, key endpoints with examples, and functionality of every code descriptively.

From the previous context, the project consists of multiple Django microservices: Auth_Service, Policy_Service, Customer_Service, Coverage_Service, Geo_Service, and Premium_Service.

The command prompt logs show creating directories and starting Django projects and apps for each service, with app names like auth_app, policy_app, etc.

In the code provided earlier, it was api for the app name, but here it's _app. Probably a variation, but the structure is similar.

I need to write a descriptive README.

Key sections:

- Introduction/Objective/Use

- Modules/Services present and their uses

- Key endpoints with examples

- Functionality of every code (probably every service's code)

Since it's a README, format it as markdown.

No tools needed here, as this is based on the provided code.

The user question includes some command prompt output, which seems like instructions on how the project was set up, but it's cut off at the end.

I think I can proceed to generate the README based on the code from the assistant's previous response.# Insurance Premium Calculator: Microservice-Based System

## Overview
This project implements a simplified insurance premium calculation platform using a microservices architecture built with Django. It transitions from a monolithic system to independent services that communicate via REST APIs. The system calculates insurance premiums based on policy details, customer risk factors, add-on coverages, and regional multipliers. All services are secured with JWT-based authentication for POST endpoints, while GET endpoints are public.

The primary use of this application is to demonstrate microservice design principles, including service orchestration, authentication, and inter-service communication. It can be used as a backend for an insurance application where administrators add policies, customers, coverages, and regions, and then compute personalized premiums.

**Key Features:**
- JWT authentication for protected operations.
- Independent services with their own databases (SQLite for simplicity).
- Premium calculation aggregates data from other services.
- Assumed premium formula (since not specified in the problem):
  - Age factor: 1.0 (<30), 1.2 (30-49), 1.5 (≥50).
  - Risk factor: 1.0 base + 0.1 (smoker) + 0.2 (has illness).
  - Final premium = (base_premium * age_factor * risk_factor * region_multiplier) + add_ons_sum.

**Tech Stack:**
- Django (for each microservice).
- Django REST Framework (for API endpoints).
- PyJWT (for JWT handling).
- Requests library (for inter-service calls in Premium Service).
- No serializers used; manual validation with custom error responses.
- ORM for database interactions.

**Setup Instructions:**
1. Install dependencies globally or in a virtual environment: `pip install django djangorestframework pyjwt requests`.
2. Create the directory structure as shown in the command logs (e.g., `mkdir Auth_Service`, etc.).
3. For each service (Auth_Service, Policy_Service, etc.):
   - Navigate to the service directory.
   - Run `django-admin startproject <service_name> .` (e.g., `auth_service`).
   - Run `python manage.py startapp <app_name>` (e.g., `auth_app` for Auth, `policy_app` for Policy, etc. – note: code uses `api` as app name, but adapt to `_app` if needed).
   - Copy the provided code snippets into the respective files (settings.py, models.py, views.py, urls.py).
   - Run migrations: `python manage.py makemigrations <app_name>; python manage.py migrate`.
4. For Auth_Service: Create a superuser `python manage.py createsuperuser` (e.g., username: admin, password: pass).
5. Run each service on different ports:
   - Auth_Service: `python manage.py runserver 8000`
   - Policy_Service: `python manage.py runserver 8001`
   - Customer_Service: `python manage.py runserver 8002`
   - Coverage_Service: `python manage.py runserver 8003`
   - Geo_Service: `python manage.py runserver 8004`
   - Premium_Service: `python manage.py runserver 8005`
6. Obtain JWT token: POST to `http://localhost:8000/login/` with JSON body `{"username": "admin", "password": "pass"}`.
7. Use the token in headers for POST requests: `Authorization: Bearer <token>`.
8. Optional: Use Docker for containerization (not implemented here, but recommended for production).

**Security Notes:**
- Shared JWT secret: 'secret' (hardcoded; use environment variables in production).
- POST endpoints require authentication; GET are public.

## Modules/Services and Their Uses
The project is divided into six independent microservices, each handling a specific domain. They use separate Django projects and apps, with their own SQLite databases. Services communicate via HTTP requests (e.g., Premium Service calls others). Below is a description of each module, its purpose, and key code functionality.

### 1. Auth_Service (Port: 8000)
   - **Purpose:** Handles user authentication and JWT token generation. It's a basic auth service using Django's built-in auth system. All other services validate JWTs issued here for protected endpoints.
   - **Use:** Authenticates admins/users before allowing data creation/modification in other services. Ensures security in a microservices setup.
   - **Key Files and Functionality:**
     - **settings.py:** Configures Django project with REST Framework, apps, middleware, and database (SQLite). Includes REST_FRAMEWORK for JSON parsing.
     - **auth_app/views.py:** Contains `login_view` function.
       - Validates required fields (username, password) manually; returns error like `{"username": ["This field is required."]}` if missing.
       - Authenticates user with Django's `authenticate()`.
       - Generates JWT token with user ID using PyJWT.
       - Returns token on success; handles invalid credentials.
     - **auth_service/urls.py:** Routes `/login/` to the view.
   - **Key Endpoints with Examples:**
     - **POST /login/**: Authenticate and get JWT.
       - Example Request (Postman): URL: `http://localhost:8000/login/`, Body (JSON): `{"username": "admin", "password": "pass"}`.
       - Example Response: `{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}`.
       - Functionality: Logs in user and issues token for use in other services.

### 2. Policy_Service (Port: 8001)
   - **Purpose:** Manages insurance policies, including adding and retrieving base premiums.
   - **Use:** Stores core policy data; used by Premium Service to fetch base premium for calculations.
   - **Key Files and Functionality:**
     - **settings.py:** Similar to Auth_Service; configures project basics.
     - **policy_app/models.py:** Defines `Policy` model with ORM fields `name` (CharField) and `base_premium` (DecimalField).
     - **policy_app/views.py:** 
       - `validate_jwt()`: Helper to check and decode JWT from headers; returns error if invalid/missing.
       - `add_policy()`: POST view; validates JWT, then required fields (name, base_premium); creates Policy instance via ORM; returns created data.
       - `get_policy()`: GET view; retrieves Policy by ID via ORM; no auth required.
     - **policy_service/urls.py:** Routes `/policies/` (POST) and `/policies/{policy_id}/` (GET).
   - **Key Endpoints with Examples:**
     - **POST /policies/**: Add a new policy (auth required).
       - Example Request: URL: `http://localhost:8001/policies/`, Headers: `Authorization: Bearer <token>`, Body: `{"name": "Health Policy", "base_premium": 1000}`.
       - Example Response: `{"id": 1, "name": "Health Policy", "base_premium": "1000.00"}` (HTTP 201).
       - Functionality: Creates policy record; validates inputs manually.
     - **GET /policies/{policy_id}/**: Retrieve policy details.
       - Example: URL: `http://localhost:8001/policies/1/`.
       - Response: `{"id": 1, "name": "Health Policy", "base_premium": "1000.00"}`.
       - Functionality: Fetches policy by ID; handles not found.

### 3. Customer_Service (Port: 8002)
   - **Purpose:** Manages customer profiles, including risk factors like age, smoker status, etc.
   - **Use:** Provides customer data for premium risk adjustments; fetched by Premium Service.
   - **Key Files and Functionality:**
     - **settings.py:** Standard Django config.
     - **customer_app/models.py:** `Customer` model with fields: name (Char), age (Int), smoker (Bool), has_illness (Bool), region (Char).
     - **customer_app/views.py:** 
       - `validate_jwt()`: Same as above.
       - `add_customer()`: Validates JWT and fields (name, age, etc.); type-checks booleans and int; creates via ORM.
       - `get_customer()`: Retrieves by ID.
     - **customer_service/urls.py:** Routes for POST/GET customers.
   - **Key Endpoints with Examples:**
     - **POST /customers/**: Add customer (auth required).
       - Example: URL: `http://localhost:8002/customers/`, Headers: Bearer <token>, Body: `{"name": "John", "age": 35, "smoker": true, "has_illness": false, "region": "Urban"}`.
       - Response: Created data (HTTP 201).
       - Functionality: Stores customer; manual validation for required fields.
     - **GET /customers/{customer_id}/**: Retrieve customer.
       - Example: `http://localhost:8002/customers/1/`.
       - Functionality: Fetches details; public access.

### 4. Coverage_Service (Port: 8003)
   - **Purpose:** Manages add-on coverages and customer selections.
   - **Use:** Handles optional coverages that add to the premium; Premium Service sums costs from selected coverages.
   - **Key Files and Functionality:**
     - **settings.py:** Standard.
     - **coverage_app/models.py:** `Coverage` (name, cost); `CustomerCoverage` (many-to-many link via customer_id and ForeignKey).
     - **coverage_app/views.py:** 
       - `validate_jwt()`: Auth check.
       - `add_coverage()`: Creates coverage.
       - `get_coverage()`: Retrieves by ID.
       - `get_customer_coverages()`: Lists coverages for a customer.
       - `add_customer_coverage()`: Extra endpoint to assign coverage to customer (required for functionality).
     - **coverage_service/urls.py:** Routes for coverages and customer coverages.
   - **Key Endpoints with Examples:**
     - **POST /coverages/**: Add coverage (auth).
       - Example: `http://localhost:8003/coverages/`, Body: `{"name": "Dental", "cost": 200}`.
       - Functionality: Creates add-on.
     - **GET /coverages/{coverage_id}/**: Retrieve coverage.
     - **POST /customers/{customer_id}/coverages/add/**: Assign coverage (auth).
       - Example: Body: `{"coverage_id": 1}`.
       - Functionality: Links coverage to customer.
     - **GET /customers/{customer_id}/coverages/**: List customer coverages.

### 5. Geo_Service (Port: 8004)
   - **Purpose:** Manages region-based premium multipliers.
   - **Use:** Provides location-specific factors; used in premium calculation.
   - **Key Files and Functionality:**
     - **settings.py:** Standard.
     - **geo_app/models.py:** `Region` (name unique, multiplier Decimal).
     - **geo_app/views.py:** 
       - `validate_jwt()`.
       - `add_region()`: Creates if not exists; checks uniqueness.
       - `get_region()`: Retrieves by name.
     - **geo_service/urls.py:** Routes for regions.
   - **Key Endpoints with Examples:**
     - **POST /regions/**: Add region (auth).
       - Example: Body: `{"name": "Urban", "multiplier": 1.1}`.
       - Functionality: Stores multiplier.
     - **GET /regions/{region_name}/**: Retrieve.
       - Example: `http://localhost:8004/regions/Urban/`.

### 6. Premium_Service (Port: 8005)
   - **Purpose:** Aggregates data from other services to compute final premium.
   - **Use:** Orchestrates calls to calculate and return premium breakdown; acts as the entry point for premium queries.
   - **Key Files and Functionality:**
     - **settings.py:** Standard.
     - No models (aggregator only).
     - **premium_app/views.py:** 
       - `validate_jwt()`.
       - `calculate_premium()`: Validates inputs; fetches data via requests to other services; computes factors and final premium; returns breakdown.
     - **premium_service/urls.py:** Route for `/premium/calculate/`.
   - **Key Endpoints with Examples:**
     - **POST /premium/calculate/**: Compute premium (auth).
       - Example: Body: `{"customer_id": 1, "policy_id": 1}`.
       - Response: `{"base_premium": 1000.0, "age_factor": 1.2, "risk_factor": 1.1, "add_ons": 200.0, "region_factor": 1.1, "final_premium": 1452.0}`.
       - Functionality: Calls services, applies formula, handles errors.

## Additional Notes
- **Error Handling:** All POSTs validate fields manually and return structured errors (e.g., HTTP 400).
- **Testing:** Use Postman collections for endpoints; start with auth token.
- **Improvements:** Add logging, error resilience, env vars, Docker-compose for orchestration.
- **Limitations:** Hardcoded URLs/ports; no production auth (use OAuth/JWT properly).
