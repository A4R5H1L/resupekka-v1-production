# System Architecture Diagram

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              USER LAYER                                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
              ┌─────▼─────┐   ┌────▼────┐   ┌─────▼─────┐
              │  Manager  │   │  User   │   │  Director │
              │  Browser  │   │ Browser │   │  Browser  │
              └───────────┘   └─────────┘   └───────────┘
                    │               │               │
                    └───────────────┼───────────────┘
                                    │
                           HTTPS (Port 443)
                                    │
┌─────────────────────────────────────────────────────────────────────────┐
│                          REVERSE PROXY LAYER                             │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │              Nginx Proxy Manager                             │      │
│  │  ┌────────────────────────────────────────────────────────┐ │      │
│  │  │  • SSL Termination (Let's Encrypt)                     │ │      │
│  │  │  • HTTPS Enforcement                                   │ │      │
│  │  │  • Request Forwarding                                  │ │      │
│  │  │  • Proxy Headers (X-Forwarded-For, X-Forwarded-Proto) │ │      │
│  │  │  • Domain: resupekka.samk.fi                           │ │      │
│  │  └────────────────────────────────────────────────────────┘ │      │
│  └──────────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                           HTTP (Internal Port 8000)
                                    │
┌─────────────────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                                │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │              Django Application (django-api)                 │      │
│  │                                                              │      │
│  │  ┌──────────────────────────────────────────────┐          │      │
│  │  │           Middleware Stack                   │          │      │
│  │  │  • CORS                                      │          │      │
│  │  │  • Security                                  │          │      │
│  │  │  • Session (4-hour timeout)                  │          │      │
│  │  │  • CSRF                                      │          │      │
│  │  │  • Authentication                            │          │      │
│  │  │  • SessionUser (permission bridge)           │          │      │
│  │  │  • SimpleHistory (audit trail)               │          │      │
│  │  └──────────────────────────────────────────────┘          │      │
│  │                        │                                    │      │
│  │  ┌──────────────────────────────────────────────┐          │      │
│  │  │           Django Apps                        │          │      │
│  │  │                                              │          │      │
│  │  │  ┌────────────┐  ┌──────────────┐          │          │      │
│  │  │  │ apps.users │  │  apps.core   │          │          │      │
│  │  │  │            │  │              │          │          │      │
│  │  │  │ • User     │  │ • Project    │          │          │      │
│  │  │  │   Model    │  │ • Role       │          │          │      │
│  │  │  │ • LDAP     │  │ • ResearchUn │          │          │      │
│  │  │  │   Sync     │  │ • ProjectSta │          │          │      │
│  │  │  └────────────┘  └──────────────┘          │          │      │
│  │  │                                              │          │      │
│  │  │  ┌──────────────┐  ┌──────────────┐        │          │      │
│  │  │  │apps.allocat  │  │apps.workflows│        │          │      │
│  │  │  │              │  │              │        │          │      │
│  │  │  │ • Contract   │  │ • WorkAlloc  │        │          │      │
│  │  │  │ • WorkAlloc  │  │   Request    │        │          │      │
│  │  │  │ • AllocType  │  │ • Approvals  │        │          │      │
│  │  │  └──────────────┘  └──────────────┘        │          │      │
│  │  │                                              │          │      │
│  │  │  ┌──────────────┐                           │          │      │
│  │  │  │  apps.api    │                           │          │      │
│  │  │  │              │                           │          │      │
│  │  │  │ • REST API   │                           │          │      │
│  │  │  │ • Endpoints  │                           │          │      │
│  │  │  └──────────────┘                           │          │      │
│  │  └──────────────────────────────────────────────┘          │      │
│  │                        │                                    │      │
│  │  ┌──────────────────────────────────────────────┐          │      │
│  │  │           Views & Templates                  │          │      │
│  │  │  • auth_views (login, logout, role select)   │          │      │
│  │  │  • dashboard_views (dashboard, projects)     │          │      │
│  │  │  • project_views (create, edit)              │          │      │
│  │  │  • logs_views (audit dashboard)              │          │      │
│  │  └──────────────────────────────────────────────┘          │      │
│  └──────────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
                     │                            │
                     │                            │
           ┌─────────▼────────┐          ┌────────▼─────────┐
           │                  │          │                  │
┌──────────▼──────────────────▼──────────▼──────────────────▼─────────────┐
│                          DATA & AUTH LAYER                               │
│                                                                          │
│  ┌─────────────────────────────┐    ┌─────────────────────────────┐   │
│  │   PostgreSQL 15 Database     │    │   SAMK LDAP Server          │   │
│  │   (db container)             │    │   (ad-ldap.samk.fi)         │   │
│  │                              │    │                              │   │
│  │  ┌────────────────────────┐ │    │  ┌────────────────────────┐ │   │
│  │  │  Tables:               │ │    │  │  • User Authentication  │ │   │
│  │  │  • users_user          │ │    │  │  • User Attributes      │ │   │
│  │  │  • core_project        │ │    │  │  • Department Info      │ │   │
│  │  │  • core_role           │ │    │  │  • Manager Hierarchy    │ │   │
│  │  │  • core_researchunit   │ │    │  │  • Single Sign-On       │ │   │
│  │  │  • allocations_contra  │ │    │  └────────────────────────┘ │   │
│  │  │  • allocations_workall │ │    │                              │   │
│  │  │  • workflows_workalloc │ │    │      LDAPS Protocol          │   │
│  │  │  • historical_*        │ │    │      (Port 636)              │   │
│  │  │    (audit tables)      │ │    │                              │   │
│  │  └────────────────────────┘ │    └─────────────────────────────┘   │
│  │                              │                                       │
│  │  Port: 5432 (internal)       │                                       │
│  │  Volume: postgres_data       │                                       │
│  └─────────────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

## Request Flow Diagram

### Standard Page Request

```
┌──────────┐
│  User    │
│  Browser │
└────┬─────┘
     │ 1. HTTPS GET https://resupekka.samk.fi/dashboard/
     │
     ▼
┌─────────────────┐
│ Nginx Proxy     │
│ Manager         │  2. Terminate SSL
│                 │  3. Add proxy headers
│                 │  4. Forward to django-api:8000
└────┬────────────┘
     │ HTTP GET /dashboard/
     │
     ▼
┌─────────────────┐
│ Django          │
│ Middleware      │  5. Process middlewares:
│ Stack           │     - Session check
│                 │     - CSRF validation
│                 │     - Auth verification
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ URL Router      │  6. Match URL pattern
│ (urls.py)       │     /dashboard/ → dashboard_view
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ View Function   │  7. Execute view logic:
│ dashboard_view  │     - Query database
│                 │     - Prepare context
└────┬────────────┘
     │
     ├──────────────────────┐
     │                      │
     ▼                      ▼
┌──────────────┐    ┌─────────────┐
│ PostgreSQL   │    │ LDAP (opt)  │
│ Query        │    │ for user    │
│              │    │ sync        │
└──────┬───────┘    └──────┬──────┘
       │                   │
       └──────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ Template        │  8. Render template
         │ Rendering       │     - dashboard.html
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ HTTP Response   │  9. HTML response
         │ (200 OK)        │
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ Nginx forwards  │  10. Return to client
         │ to browser      │
         └────┬────────────┘
              │
              ▼
         ┌──────────┐
         │  Browser │  11. Render page
         │  Display │
         └──────────┘
```

### LDAP Authentication Flow

```
┌──────────┐
│  User    │  1. Submit login form
│  Browser │     username + password
└────┬─────┘
     │
     ▼
┌─────────────────┐
│ Nginx Proxy     │  2. Forward to Django
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ auth_views.py   │  3. samk_login view
│ samk_login()    │     receives credentials
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ apps.core.auth  │  4. authenticate_ldap()
│ LDAP Handler    │
└────┬────────────┘
     │
     ├────────────────────────────────────────┐
     │                                        │
     ▼                                        ▼
┌──────────────┐                    ┌─────────────────┐
│ LDAP Server  │  5. Bind with      │ PostgreSQL      │
│ ad-ldap.samk │     credentials    │                 │
│              │  6. Fetch user     │  8. Query User  │
│              │     attributes     │     by email    │
└──────┬───────┘                    └──────┬──────────┘
       │                                   │
       │ 7. Return user data               │
       │    (name, email, dept)            │
       └──────────┬────────────────────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ Create/Update   │  9. User.objects.update_or_create()
         │ User in DB      │     Sync LDAP data to database
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ Create Session  │  10. session['samk_authenticated'] = True
         │                 │      session['user_id'] = user.id
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ Redirect to     │  11. Redirect /select-role/
         │ Role Selection  │
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ User selects    │  12. User, Manager, Admin, Director
         │ role            │      session['samk_role'] = selected_role
         └────┬────────────┘
              │
              ▼
         ┌─────────────────┐
         │ Redirect to     │  13. Redirect /dashboard/
         │ Dashboard       │      User now authenticated and authorized
         └─────────────────┘
```

## Component Details

### 1. Nginx Proxy Manager

**Purpose:** Public-facing reverse proxy with SSL termination

**Responsibilities:**
- Accept HTTPS requests on port 443
- SSL/TLS termination using Let's Encrypt certificates
- HTTP to HTTPS redirection (port 80 → 443)
- Forward requests to Django application (django-api:8000)
- Add proxy headers for Django CSRF and security

**Key Headers:**
```
X-Forwarded-For: $proxy_add_x_forwarded_for
X-Forwarded-Proto: $scheme
X-Forwarded-Host: $host
X-Real-IP: $remote_addr
```

**Network:** External nginx-proxy network + host network (ports 80/443)

### 2. Django Application

**Purpose:** Core business logic and web application

**Components:**

**Middleware Stack (Order Matters):**
1. CORS - Cross-origin support
2. Security - Django security headers
3. Session - Session management with 4-hour timeout
4. CSRF - Cross-site request forgery protection
5. Authentication - Django auth system
6. SessionUser - Custom middleware bridging session to Django permissions
7. Messages - Flash messages
8. SimpleHistory - Audit trail tracking

**Django Apps:**
- **apps.users** - User management, custom User model
- **apps.core** - Projects, Roles, ResearchUnits, ProjectStatus
- **apps.allocations** - Contracts, WorkAllocations, AllocationTypes
- **apps.workflows** - Approval workflows and requests
- **apps.api** - REST API endpoints

**URL Patterns:**
- `/admin/` - Django admin interface
- `/login/`, `/logout/`, `/select-role/` - Authentication
- `/dashboard/` - Main dashboards
- `/project/new/` - Project creation
- `/api/` - REST API

**Network:** nginx-proxy (external) + internal networks

### 3. PostgreSQL Database

**Purpose:** Primary data persistence

**Databases:**
- resupekka_db - Main application database

**Key Tables:**
- User tables (users_user + historical)
- Core tables (projects, roles, research units)
- Allocation tables (contracts, work allocations)
- Workflow tables (requests)
- Historical tables (audit trail via django-simple-history)

**Connection:**
- Django connects via database URL
- psycopg2 adapter
- Port 5432 (internal only)

**Volume:** postgres_data (persistent storage)

**Network:** internal network only

### 4. LDAP Server

**Purpose:** Authentication and user data source

**Server:** ad-ldap.samk.fi (SAMK Active Directory)
**Protocol:** LDAPS (LDAP over SSL, port 636)

**Operations:**
- Bind (authentication)
- Search (user lookup)
- Attribute retrieval (name, email, department, manager)

**Integration:**
- Custom authentication in Django
- User synchronization on login
- Manager hierarchy mapping

**Network:** External university network

### 5. MCP Server (Supporting Service)

**Purpose:** Model Context Protocol server for extended features

**Port:** 8001 (internal)
**Network:** internal network only
**Connection:** Communicates with Django API

## Network Architecture

### Docker Networks

**nginx-proxy (external):**
- Created by Nginx Proxy Manager
- Bridge network
- Services: Nginx Proxy Manager, django-api, db
- Purpose: Allow proxy to route to Django

**internal (bridge):**
- Created by docker-compose.yml
- Services: All services (django-api, db, mcp-server, pgadmin)
- Purpose: Internal service communication
- Isolated from public internet

### Port Mapping

**Public-facing:**
- 80/tcp → Nginx (HTTP, redirects to 443)
- 443/tcp → Nginx (HTTPS)

**Localhost-only:**
- 127.0.0.1:8080 → PgAdmin (admin tool)

**VPN-only:**
- 81/tcp → Nginx Proxy Manager admin UI
- 22/tcp → SSH server

**Internal-only (no external access):**
- 8000/tcp → Django application
- 5432/tcp → PostgreSQL database
- 8001/tcp → MCP server

## Data Flow Patterns

### User Login Flow

```
User → Nginx → Django → LDAP → Validate
                  ↓
              PostgreSQL ← Create/Update User
                  ↓
              Session Created → Role Selection → Dashboard
```

### Dashboard Page Load

```
User → Nginx → Django → Session Check → Auth Check
                  ↓
              Query PostgreSQL (projects, allocations, users)
                  ↓
              Render Template → HTML Response
                  ↓
              Nginx → User (display dashboard)
```

### Project Creation

```
User → Form Submit → Nginx → Django
                              ↓
                         CSRF Validation
                              ↓
                         Auth Check (role = Admin/Director)
                              ↓
                         Create Project in PostgreSQL
                              ↓
                         SimpleHistory logs change
                              ↓
                         Redirect to Dashboard
```

### Allocation Change Approval

```
Manager → Request Change → Django
                            ↓
                       Create WorkAllocationRequest
                            ↓
                       Generate access token
                            ↓
                       Send email to approver

Approver → Click link → Django → Verify token
                                    ↓
                                Update allocation
                                    ↓
                                Mark request approved
                                    ↓
                                SimpleHistory logs change
```

## Security Architecture

### Security Layers

**Layer 1: Network Security**
- Firewall rules (iptables/ufw)
- Docker network isolation
- Only Nginx exposed to public

**Layer 2: SSL/TLS**
- Let's Encrypt certificates
- HTTPS enforcement
- Nginx SSL termination

**Layer 3: Application Security**
- CSRF protection
- SQL injection prevention (ORM)
- XSS protection (template escaping)
- Clickjacking protection

**Layer 4: Authentication**
- LDAP authentication (university credentials)
- Session-based auth (4-hour timeout)
- No passwords stored in Django database

**Layer 5: Authorization**
- Role-based access control
- Session role storage
- View-level permission checks

**Layer 6: Audit Trail**
- django-simple-history tracks all changes
- Who, what, when for every modification
- Immutable audit log

## Scalability Architecture

### Current Scale

**Single Server:**
- All services on one Docker host
- Vertical scaling available
- Adequate for 200+ users

### Horizontal Scaling Path

```
                     ┌──────────────┐
                     │ Load Balancer│
                     └──────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   ┌────▼─────┐       ┌────▼─────┐       ┌────▼─────┐
   │ Django 1 │       │ Django 2 │       │ Django 3 │
   └────┬─────┘       └────┬─────┘       └────┬─────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                     ┌──────▼────────┐
                     │  PostgreSQL   │
                     │  (with replicas)│
                     └───────────────┘
```

### Caching Layer (Future)

```
User → Nginx → Django → Redis Cache → PostgreSQL
                          │              │
                          └──────────────┘
                          (cache miss)
```

## Deployment Architecture

### Container Orchestration

```
docker-compose.yml defines:
  ├── django-api (web application)
  ├── db (PostgreSQL)
  ├── mcp-server (support service)
  └── pgadmin (admin tool)

Separate Nginx Proxy Manager stack
```

### Volume Management

```
Volumes:
  ├── postgres_data → /var/lib/postgresql/data (persistent database)
  ├── django_static → /app/staticfiles (static files)
  └── /var/log/resupekka → application logs
```

## Monitoring Architecture (Current + Future)

### Current Monitoring

```
Docker healthcheck → PostgreSQL (pg_isready)
Container status → docker-compose ps
Application logs → docker-compose logs
```

### Future Monitoring

```
                 ┌─────────────┐
                 │  Prometheus │  ← Metrics collection
                 └──────┬──────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
   │ Django  │    │   DB    │    │  Nginx  │
   │ Metrics │    │ Metrics │    │ Metrics │
   └─────────┘    └─────────┘    └─────────┘
        │
        ▼
   ┌─────────┐
   │ Grafana │  ← Dashboards
   └─────────┘

   ┌─────────┐
   │ Sentry  │  ← Error tracking
   └─────────┘
```

## Conclusion

Resupekka's architecture demonstrates:

**Simplicity:** Three-tier architecture (web, app, data)
**Security:** Multiple layers of protection
**Maintainability:** Clear separation of concerns
**Scalability:** Vertical scaling now, horizontal scaling path available
**Reliability:** Stable for 4+ months serving real users

The architecture is appropriate for the current scale and provides a solid foundation for future growth.
