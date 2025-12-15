# System Architecture

## Overview

Resupekka V1 is a Django-based web application following a traditional three-tier architecture pattern. The system manages work allocations, project tracking, and personnel contracts for research operations at SAMK.

## System Components

### 1. Nginx Proxy Manager (Reverse Proxy Layer)

**Role:** Public-facing gateway and SSL termination

**Responsibilities:**
- SSL/TLS certificate management via Let's Encrypt
- Reverse proxy routing to Django application
- HTTPS enforcement and HTTP to HTTPS redirection
- Request forwarding with proper proxy headers
- Static file serving (if configured)

**Configuration:**
- Domain: `resupekka.samk.fi`
- Ports: 80 (HTTP) → 443 (HTTPS)
- Admin interface: Port 81 (VPN-only access)
- Automated certificate renewal

**Network Security:**
- External nginx-proxy Docker network
- Firewall rules: ports 80/443 open from STAFF networks only
- Internal services not exposed to public internet

### 2. Django Application (Business Logic Layer)

**Role:** Core application server and business logic

**Framework:** Django 5.2.6 with Django REST Framework 3.16.0

**Application Structure:**

```
src/
├── resupekka/              # Project configuration
│   ├── settings/           # Environment-specific settings
│   │   ├── base.py        # Common settings
│   │   └── development.py # Development overrides
│   ├── urls.py            # URL routing
│   ├── wsgi.py            # WSGI entry point
│   └── asgi.py            # ASGI entry point
│
├── apps/
│   ├── users/             # User management and authentication
│   ├── core/              # Core models (Projects, ResearchUnits, Roles)
│   ├── allocations/       # Work allocations and contracts
│   ├── workflows/         # Approval workflows
│   └── api/               # REST API endpoints
│
├── templates/             # Django templates
└── static/               # Static assets (CSS, JS, images)
```

**Django Apps:**

1. **apps.users** - Custom user model with LDAP integration
2. **apps.core** - Project management, research units, roles, project status
3. **apps.allocations** - Work allocations, contracts, allocation types
4. **apps.workflows** - Request approval workflows with token-based approval
5. **apps.api** - REST API infrastructure (DRF endpoints ready for future frontend framework migration)

**Middleware Stack:**
- CORS middleware (cross-origin support)
- Security middleware (Django security features)
- Session middleware (4-hour expiration via custom SessionExpirationMiddleware)
- CSRF middleware (with reverse proxy configuration)
- Authentication middleware (Django auth + custom SessionUserMiddleware)
- Simple History middleware (audit trail tracking)

**Current Deployment Mode:**
- Running via `python manage.py runserver 0.0.0.0:8000`
- Development mode with DEBUG=True
- Suitable for internal use but can be upgraded to production-grade setup

### 3. PostgreSQL Database (Data Persistence Layer)

**Role:** Primary data store

**Version:** PostgreSQL 15
**Connection:** Via psycopg2-binary 2.9.10
**Database Name:** resupekka_db

**Database Features:**
- Relational data model with foreign key constraints
- Unique constraints for data integrity
- Historical tables via django-simple-history
- Connection pooling (default Django database pooling)

**Backup Considerations:**
- PostgreSQL data stored in Docker volume: `postgres_data`
- Database accessible internally on port 5432
- PgAdmin available for administration on localhost:8080

### 4. LDAP Server (Authentication Layer)

**Role:** Centralized authentication and user data source

**LDAP Server:** `ad-ldap.samk.fi` (SAMK Active Directory)
**Protocol:** LDAPS (LDAP over SSL)

**Integration:**
- Custom LDAP authentication in `apps.core.auth`
- User data synchronized from LDAP directory
- Session-based authentication (not using django-auth-ldap backend)
- Fields imported: first_name, last_name, email, department, company, manager

**Authentication Flow:**
1. User enters credentials on login page
2. Django validates against LDAP server
3. User object created/updated in local database
4. Session established with role selection
5. Session middleware validates on each request

### 5. MCP Server (Support Service)

**Role:** Model Context Protocol server for additional functionality

**Port:** 8001 (internal)
**Purpose:** Supplementary service for extended features
**Network:** Internal Docker network only

## Request Flow

### Standard User Request Flow

```
1. User Browser
   ↓ HTTPS request to resupekka.samk.fi

2. Nginx Proxy Manager (Port 443)
   ↓ SSL termination
   ↓ Add proxy headers (X-Forwarded-For, X-Forwarded-Proto)
   ↓ Forward to django-api:8000

3. Django Application
   ↓ CORS middleware
   ↓ Security middleware
   ↓ Session middleware (check 4-hour expiration)
   ↓ CSRF validation
   ↓ Authentication middleware (check session)
   ↓ Custom SessionUserMiddleware (bridge session to Django permissions)
   ↓ URL routing (urls.py)
   ↓ View function/class
   ↓ Business logic
   ↓ Database query (if needed)

4. PostgreSQL Database
   ↓ Execute query
   ↓ Return results

5. Django Response
   ↓ Render template or JSON
   ↓ Return HTTP response

6. Nginx Proxy Manager
   ↓ Forward response to browser

7. User Browser
   ↓ Display page/data
```

### LDAP Authentication Flow

```
1. User submits login form
   ↓ POST /login/ with username/password

2. Django auth view (samk_login)
   ↓ apps.core.auth.authenticate_ldap()
   ↓ Connect to ad-ldap.samk.fi
   ↓ Bind with user credentials

3. LDAP Server
   ↓ Validate credentials
   ↓ Return user attributes

4. Django auth handler
   ↓ Create/update User object in database
   ↓ Set session['samk_authenticated'] = True
   ↓ Store user_id in session
   ↓ Redirect to /select-role/

5. Role selection
   ↓ User selects role (User/Manager/Admin/Director)
   ↓ Set session['samk_role']
   ↓ Redirect to /dashboard/
```

## Django Application Structure

### URL Routing Pattern

**Main routes** (from `src/resupekka/urls.py`):

- `/admin/` - Django admin interface
- `/login/` - LDAP authentication
- `/logout/` - Session termination
- `/select-role/` - Role selection after login
- `/dashboard/` - Main dashboard (role-based view)
- `/dashboard/projects/` - Project overview
- `/dashboard/audit/` - Audit log viewer
- `/project/new/` - Create new project
- `/api/` - REST API endpoints

**Dashboard actions:**
- Edit allocation, contract, project
- Add/remove persons from projects
- Change project status
- Approve/reject workflow requests

### View Organization

Views are organized by functionality in `apps.core.views/`:

1. **auth_views.py** - Authentication (login, logout, role selection)
2. **dashboard_views.py** - Dashboard displays and actions
3. **project_views.py** - Project creation and management
4. **logs_views.py** - Audit trail viewing

### Model Relationships

See [Database Schema diagram](../diagrams/database-schema.md) for detailed entity relationships.

**Core entities:**
- User (custom model extending AbstractUser)
- Role (User, Manager, Admin, Director)
- ResearchUnit (TKI, RoboAI, etc.)
- Project (research projects with date ranges)
- ProjectStatus (Active, Finished, BeingPrepared, etc.)
- Contract (user-role-unit assignments)
- WorkAllocation (monthly percentage allocations)
- WorkAllocationType (Normal, Flat Rate)
- WorkAllocationRequest (approval workflow)

## Database Design

### Core Tables

**users_user**
- Custom user model (email-based authentication)
- LDAP-synchronized fields
- Role assignments through sessions (not database field)
- Manager preferences for dashboard filtering

**core_project**
- Project metadata and lifecycle
- Links to ResearchUnit and ProjectStatus
- Date range tracking (start_date, end_date, end_date_original)
- Notification email configuration
- Audit history via django-simple-history

**allocations_contract**
- Personnel contracts linking User → Role → ResearchUnit
- Date range and work percentage
- Foundation for work allocations

**allocations_workallocation**
- Monthly work allocations
- Links Contract → Project → WorkAllocationType
- Percentage-based (0-100%)
- Audit history tracking

**workflows_workallocationrequest**
- Approval workflow for allocation changes
- Token-based approval system
- Status tracking (pending, approved, rejected)

### Data Integrity

**Unique constraints:**
- Contract: user + role + unit + start_date + end_date
- WorkAllocation: user + project + allocation_type + month
- User: email (primary identifier)

**Foreign key restrictions:**
- ON DELETE RESTRICT prevents accidental data loss
- Ensures referential integrity across all relationships

### Audit Trail

**django-simple-history integration:**
- Historical records for: User, Project, WorkAllocation, WorkAllocationRequest
- Tracks: what changed, who changed it, when it changed
- Accessible via `.history.all()` on model instances
- Middleware captures request user for audit context

## LDAP Integration Details

### Authentication Architecture

**Custom implementation** (not using django-auth-ldap backend):
- Manual LDAP bind in `apps.core.auth`
- Session-based authentication
- User synchronization on each login

**Why custom approach:**
- Greater control over authentication flow
- Easier role selection workflow
- Simpler session management
- No dependency on django-auth-ldap's ModelBackend

### User Synchronization

**On each login:**
1. LDAP bind validates credentials
2. Extract user attributes from LDAP
3. Query User model by email
4. Create new user or update existing user
5. Sync: first_name, last_name, email, department, company, manager

**LDAP fields mapped:**
- givenName → first_name
- sn → last_name
- mail → email
- department → department
- company → company
- manager DN → manager_email/manager_username

### Permission Mapping

**Role-based access:**
- Roles stored in session, not database
- SessionUserMiddleware bridges session to Django permissions
- Views check session['samk_role'] for authorization
- Admin/Director roles have elevated privileges

## Infrastructure Deployment

### Docker Compose Services

**Service: django-api**
- Build: Custom Dockerfile (docker/Dockerfile.django)
- Command: `python manage.py runserver 0.0.0.0:8000`
- Volumes: Source code mount, static files, logs
- Environment: DEBUG=True, development settings
- Network: nginx-proxy (external), internal

**Service: db**
- Image: postgres:15
- Volumes: postgres_data (persistent storage)
- Healthcheck: pg_isready
- Network: nginx-proxy (for pgadmin), internal

**Service: mcp-server**
- Build: Custom Dockerfile (docker/Dockerfile.mcp)
- Network: internal only

**Service: pgadmin**
- Image: dpage/pgadmin4
- Port: 127.0.0.1:8080 (localhost only)
- Network: internal

### Network Topology

**nginx-proxy network:**
- External network (created by Nginx Proxy Manager)
- Services: django-api, db
- Purpose: Allows proxy to route to Django

**internal network:**
- Bridge network
- Services: all services
- Purpose: Internal communication, no external exposure

### Security Hardening

**Network isolation:**
- Only Nginx exposed to public internet (ports 80/443)
- Application and database on internal networks
- PgAdmin restricted to localhost

**CSRF configuration:**
- CSRF_TRUSTED_ORIGINS includes resupekka.samk.fi
- CSRF_COOKIE_SECURE = False (for development mode)
- Proxy headers properly configured

**Session security:**
- 4-hour session timeout (SessionExpirationMiddleware)
- SESSION_COOKIE_SECURE = False (development)
- Can be upgraded to secure cookies for production

## Scalability Considerations

### Current Architecture

**Single-server deployment:**
- All services on one Docker host
- Successfully serving 582 users across 5 departments
- Handles 5,948 allocations and 57 projects effectively
- Vertical scaling possible (increase server resources)

### Potential Improvements

**For increased load:**

1. **Application server upgrade:**
   - Replace `runserver` with Gunicorn/uWSGI
   - Multiple worker processes
   - Process manager (systemd/supervisor)

2. **Static file optimization:**
   - Dedicated static file server
   - CDN for static assets
   - Django whitenoise for simple static serving

3. **Database optimization:**
   - Connection pooling (pgbouncer)
   - Read replicas for reporting
   - Database query optimization

4. **Caching layer:**
   - Redis for session storage
   - Cached database queries
   - Template fragment caching

5. **Horizontal scaling:**
   - Load balancer (HAProxy/Nginx)
   - Multiple Django application servers
   - Shared session storage (Redis/database)

### Current Performance

**Deployment status:**
- Stable for 4+ months
- Handles concurrent user sessions
- No reported performance issues
- Development server adequate for current scale

## Monitoring and Logging

### Current Logging

**Django logging:**
- Console output for LDAP authentication
- Log level: INFO for apps.core.auth
- Development logging configuration

**Nginx logs:**
- Access logs in nginx-proxy-manager/data/logs/
- Error logs for troubleshooting

### Recommendations for Production

**Application monitoring:**
- Django logging to file
- Error tracking (Sentry or similar)
- Performance monitoring (Django Debug Toolbar in dev)

**Infrastructure monitoring:**
- Docker container health checks
- PostgreSQL query performance
- Disk space monitoring
- SSL certificate expiration alerts

## Disaster Recovery

### Current Backup Strategy

**Database:**
- PostgreSQL data in Docker volume
- Manual backups recommended

**Application code:**
- Version controlled (Git)
- Separate from deployment documentation

### Recommended Improvements

1. **Automated database backups**
   - Daily pg_dump to external storage
   - Retention policy (30 days)
   - Test restore procedures

2. **Volume backups**
   - Docker volume snapshots
   - Nginx configuration backup

3. **Documentation**
   - Runbook for restore procedures
   - Contact information for emergencies

## Development Workflow

**Local development:**
- SQLite or local PostgreSQL
- Development settings with DEBUG=True
- Mock LDAP or test credentials

**Deployment:**
- Docker Compose on production server
- Environment variables via .env file
- PostgreSQL migrations applied manually

**Version control:**
- Git for source code
- Feature branches for development
- Direct deployment to production (single-person team)

## Technology Choices

### Why Django?

- Batteries-included framework
- Built-in admin interface
- ORM for database abstraction
- Strong security defaults
- Excellent documentation

### Why PostgreSQL?

- Robust relational database
- Django ORM compatibility
- ACID compliance for data integrity
- Good performance for read-heavy workloads

### Why Docker Compose?

- Simple orchestration for multi-service app
- Reproducible environments
- Easy local development setup
- Suitable for single-server deployment

### Why Custom LDAP Integration?

- Specific authentication flow requirements
- Role selection workflow unique to Resupekka
- Session-based rather than backend-based auth
- Easier debugging and customization

## Security Architecture

### Authentication Layers

1. **LDAP authentication** - Validates credentials against SAMK AD
2. **Django sessions** - Maintains logged-in state
3. **Role-based authorization** - Controls feature access
4. **CSRF protection** - Prevents cross-site attacks

### Data Protection

- **In transit:** HTTPS via Let's Encrypt
- **At rest:** PostgreSQL database (can add encryption)
- **Session data:** Django session framework (database-backed)
- **Audit trails:** django-simple-history tracks all changes

### Access Control

**Four-tier role system:**
1. **User** - View own allocations and projects
2. **Manager** - Manage team allocations within research unit
3. **Admin** - Cross-unit administrative access
4. **Director** - Full system access

**Implemented via:**
- Session-based role storage
- View decorators checking session['samk_role']
- Template conditionals for UI elements

## API Architecture

**Django REST Framework integration:**
- Token authentication available
- Session authentication for web UI
- Pagination: 50 items per page
- Filtering via django-filter

**API endpoints:**
- Located in apps/api/urls.py
- RESTful design patterns
- JSON responses
- Used for AJAX operations in dashboard

## Future Architecture Enhancements

**Potential improvements:**

1. **Production-grade WSGI server** (Gunicorn)
2. **Dedicated static file server** (Nginx or whitenoise)
3. **Caching layer** (Redis)
4. **Celery for background tasks** (email notifications, reports)
5. **Read replicas** (if reporting load increases)
6. **Containerized static file builds** (if frontend complexity grows)
7. **Health check endpoints** (for monitoring)
8. **Prometheus metrics export** (for observability)

**Note:** Current architecture is appropriate for the scale and serves users effectively. Enhancements should be driven by actual performance needs or user growth.
