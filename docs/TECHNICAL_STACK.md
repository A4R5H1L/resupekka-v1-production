# Technical Stack

## Overview

Resupekka V1 is built on a modern Python/Django stack with PostgreSQL database and Docker-based deployment. The technology choices prioritize stability, maintainability, and integration with university infrastructure.

## Backend Framework

### Django 5.2.6

**Web Framework**
- **Version:** 5.2.6 (latest LTS series)
- **Purpose:** Core web framework and application structure
- **Key Features Used:**
  - Django ORM for database abstraction
  - Built-in admin interface
  - Template engine for HTML rendering
  - Forms and validation
  - Authentication framework
  - Middleware system
  - Management commands

**Why Django:**
- Batteries-included philosophy (less third-party dependencies)
- Strong security defaults (CSRF, SQL injection protection)
- Excellent documentation and community support
- Built-in admin interface saves development time
- ORM simplifies database operations
- Proven track record for database-heavy applications

### Django REST Framework 3.16.0

**API Framework**
- **Version:** 3.16.0
- **Purpose:** RESTful API endpoints
- **Features Used:**
  - Serializers for data transformation
  - ViewSets for API views
  - Token authentication
  - Session authentication
  - Pagination (50 items per page)
  - API browsable interface

**API Capabilities:**
- JSON responses for AJAX operations
- Token-based authentication for external integrations
- Session authentication for web UI
- Filtering and pagination
- RESTful endpoint design

### Python

**Language Version:** Python 3.x (likely 3.10 or 3.11 based on Django 5.2.6 requirements)

**Standard Library Usage:**
- pathlib for path operations
- datetime for date/time handling
- os for environment variables
- logging for application logs

**Python Packages:** See Dependencies section below

## Database

### PostgreSQL 15

**Relational Database**
- **Version:** 15 (latest stable at deployment)
- **Purpose:** Primary data persistence
- **Deployment:** Docker container

**Features Used:**
- ACID transactions
- Foreign key constraints
- Unique constraints
- Indexes (Django automatic indexing)
- Connection pooling (Django default)

**Database Characteristics:**
- Single database: `resupekka_db`
- User: `resupekka_user`
- Encoding: UTF-8
- Timezone support: Europe/Helsinki

**Why PostgreSQL:**
- Robust and reliable
- Excellent Django support
- ACID compliance for data integrity
- Good performance for read-heavy workloads
- JSON field support (for future features)
- Strong constraint enforcement

### psycopg2-binary 2.9.10

**Database Adapter**
- **Version:** 2.9.10
- **Purpose:** PostgreSQL database driver for Python
- **Type:** Binary package (includes compiled C extension)

**Connection Management:**
- Django's database connection pooling
- Automatic connection management per request
- Transaction support via Django ORM

## Authentication & Security

### django-auth-ldap 4.8.0

**LDAP Integration Library**
- **Version:** 4.8.0
- **Purpose:** LDAP authentication support
- **Note:** Package installed but custom auth handler used

**Integration Approach:**
- Custom authentication in `apps.core.auth`
- Direct LDAP binding (not using django-auth-ldap backend)
- Session-based authentication
- Manual user synchronization

### python-ldap 3.4.4

**LDAP Client Library**
- **Version:** 3.4.4
- **Purpose:** Low-level LDAP protocol communication
- **Usage:** LDAP connection and query operations

**LDAP Configuration:**
- **Server:** ad-ldap.samk.fi
- **Protocol:** LDAPS (LDAP over SSL)
- **Operations:** Bind, search, attribute retrieval

**Security Features:**
- SSL/TLS encrypted LDAP connection
- Credential validation against SAMK Active Directory
- User attribute synchronization

### Security Libraries

**Django Built-in Security:**
- CSRF protection (django.middleware.csrf.CsrfViewMiddleware)
- SQL injection prevention (parameterized queries)
- XSS protection (template auto-escaping)
- Clickjacking protection (X-Frame-Options)
- Secure password hashing (PBKDF2)

**SSL/TLS:**
- Let's Encrypt certificates via Nginx Proxy Manager
- HTTPS enforcement at proxy layer

## Supporting Django Packages

### django-simple-history 3.7.0

**Audit Trail System**
- **Version:** 3.7.0
- **Purpose:** Automatic change tracking for models
- **Features:**
  - Historical records for all changes
  - Who made the change (user tracking)
  - When the change occurred (timestamp)
  - What changed (field-level tracking)

**Models with History:**
- User (custom user model)
- Project
- WorkAllocation
- WorkAllocationRequest

**Implementation:**
```python
from simple_history.models import HistoricalRecords

class Project(models.Model):
    # ... fields ...
    history = HistoricalRecords()
```

**Middleware:**
- `simple_history.middleware.HistoryRequestMiddleware`
- Captures request user for audit context

### django-extensions 4.1

**Development Utilities**
- **Version:** 4.1
- **Purpose:** Enhanced management commands and debugging tools
- **Features Used:**
  - shell_plus (enhanced Django shell)
  - runserver_plus (enhanced development server)
  - Additional management commands

**Development Benefits:**
- Faster debugging and exploration
- Better REPL experience
- Useful utility commands

### django-filter 24.2

**Query Filtering**
- **Version:** 24.2
- **Purpose:** Dynamic filtering for querysets
- **Integration:** REST Framework filtering backend

**Usage:**
- API endpoint filtering
- Dashboard query parameters
- Dynamic report generation

### django-cors-headers 4.4.0

**CORS Management**
- **Version:** 4.4.0
- **Purpose:** Cross-Origin Resource Sharing configuration
- **Middleware:** `corsheaders.middleware.CorsMiddleware`

**Configuration:**
```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:8001",
    "http://127.0.0.1:8001",
]
```

**Use Case:**
- MCP server communication
- Future frontend integrations
- AJAX requests

## Configuration Management

### python-decouple 3.8

**Environment Variable Management**
- **Version:** 3.8
- **Purpose:** Separate configuration from code
- **Features:**
  - .env file support
  - Type casting (bool, int, list)
  - Default values
  - Environment variable override

**Usage Example:**
```python
from decouple import config

DEBUG = config('DEBUG', default=True, cast=bool)
SECRET_KEY = config('SECRET_KEY', default='dev-key')
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])
```

**Benefits:**
- Secrets not in source code
- Different configs for dev/prod
- Easy deployment configuration

### dj-database-url 2.2.0

**Database URL Parsing**
- **Version:** 2.2.0
- **Purpose:** Parse database connection strings
- **Format:** `postgresql://user:password@host:port/database`

**Usage:**
```python
import dj_database_url

DATABASE_URL = config('DATABASE_URL')
DATABASES = {'default': dj_database_url.parse(DATABASE_URL)}
```

**Benefits:**
- 12-factor app compliance
- Easy environment switching
- Standard database URL format

## Utility Libraries

### python-dateutil 2.9.0

**Date/Time Utilities**
- **Version:** 2.9.0
- **Purpose:** Advanced date/time parsing and manipulation
- **Features:**
  - Relative date calculations
  - Timezone handling
  - Date parsing from various formats

**Use Cases:**
- Allocation date range calculations
- Project timeline operations
- Monthly allocation processing

## Infrastructure

### Docker

**Container Platform**
- **Purpose:** Application packaging and deployment
- **Orchestration:** Docker Compose

**Containers:**
1. **django-api** - Main Django application
2. **db** - PostgreSQL 15 database
3. **mcp-server** - MCP support service
4. **pgadmin** - Database administration

**Docker Compose Version:** 3.x syntax

**Benefits:**
- Reproducible environments
- Service isolation
- Easy deployment and scaling
- Consistent dev/prod setup

### Nginx Proxy Manager

**Reverse Proxy & SSL Manager**
- **Purpose:** Public-facing gateway with SSL termination
- **Features:**
  - Let's Encrypt integration
  - SSL certificate automation
  - Reverse proxy configuration
  - Web-based management UI

**Configuration:**
- Domain: resupekka.samk.fi
- SSL: Let's Encrypt automated certificates
- Proxy: Forward to django-api:8000

**Benefits:**
- Automatic SSL certificate renewal
- Easy proxy configuration
- Professional SSL setup
- No manual certificate management

### PostgreSQL Docker Image

**Database Container**
- **Image:** postgres:15 (official Docker image)
- **Volume:** postgres_data (persistent storage)
- **Healthcheck:** pg_isready command

## Frontend Technologies

### Django Templates

**Template Engine**
- **Engine:** Django Template Language (DTL)
- **Location:** src/templates/
- **Features:**
  - Template inheritance
  - Context processors
  - Template tags and filters
  - Auto-escaping for XSS protection

**Template Structure:**
- Base templates for consistent layout
- Component templates for reusable elements
- View-specific templates for pages

### Static Files

**CSS:**
- Custom stylesheets
- SAMK branding styles
- Dashboard layout CSS

**JavaScript:**
- Dashboard interactivity
- AJAX requests for dynamic updates
- Form validation
- No heavy frontend framework (vanilla JS)

**Images:**
- SAMK logo and branding
- Campus background images
- UI icons and graphics

**Static File Management:**
- `django.contrib.staticfiles` app
- collectstatic command for deployment
- Static file serving via Django development server

### No Frontend Framework

**Deliberate Choice:**
- Server-rendered HTML (Django templates)
- Progressive enhancement with JavaScript
- No React/Vue/Angular complexity
- Fast time-to-interactive
- SEO-friendly (server-rendered)

**Benefits:**
- Simpler architecture
- Less build tooling
- Faster development for solo developer
- Easier maintenance
- No JavaScript framework versioning issues

## Development Tools

### Version Control

**Git**
- Source code version control
- Feature branches for development
- Deployment via git pull

**Repository Structure:**
- Separate repository for source code (proprietary)
- This documentation repository (architecture only)

### Dependency Management

**pip & requirements files**
- **Base:** requirements/base.txt
- **Development:** requirements/development.txt
- **MCP:** requirements/mcp.txt

**Installation:**
```bash
pip install -r requirements/base.txt
```

**Dependency Pinning:**
- Exact versions specified (e.g., Django==5.2.6)
- Ensures consistent environments
- Security updates via manual version bumps

## Deployment Stack

### Operating System

**Ubuntu Linux**
- Server environment
- SSH access for deployment
- systemd for service management (potential)

### Web Server

**Development Server (Current):**
- `python manage.py runserver 0.0.0.0:8000`
- Django built-in development server
- Adequate for current load

**Production Recommendation:**
- **Gunicorn** (Green Unicorn)
- **uWSGI** (alternative)
- **Multiple worker processes**
- **Process manager (systemd/supervisor)**

### Process Management

**Current:** Docker restart policy (unless-stopped)

**Recommended for Production:**
- systemd service file
- Supervisor process manager
- Automatic restart on failure
- Log rotation

### Log Management

**Current Logging:**
- Docker container logs
- Django console logging
- Nginx access/error logs

**Log Locations:**
- Application: `/var/log/resupekka/`
- Nginx: `nginx-proxy-manager/data/logs/`
- PostgreSQL: Docker logs

### Monitoring

**Health Checks:**
- PostgreSQL: pg_isready
- Docker: container status

**Potential Additions:**
- Application health endpoint
- Prometheus metrics export
- Sentry error tracking
- Uptime monitoring

## Network Architecture

### Docker Networks

**nginx-proxy (external):**
- Bridge network
- Created by Nginx Proxy Manager
- Allows proxy to route to Django

**internal (bridge):**
- Internal Docker network
- Service-to-service communication
- Not exposed to public internet

### Firewall

**iptables/ufw**
- Ports 80/443: HTTPS access (STAFF networks)
- Port 81: Nginx admin (VPN only)
- Port 22: SSH (VPN only)
- All other ports blocked

## Data Format Standards

### Date/Time

**Format:** ISO 8601
- Date fields: YYYY-MM-DD
- Datetime fields: YYYY-MM-DD HH:MM:SS
- Timezone: Europe/Helsinki

**Django Settings:**
```python
TIME_ZONE = 'Europe/Helsinki'
USE_TZ = True
DATE_FORMAT = 'd/m/Y'
```

### Percentage Representation

**Storage:** Float (0-100)
- 20% stored as 20.0 (not 0.2)
- Validators: MinValueValidator(0), MaxValueValidator(100)

### JSON API

**REST Framework:**
- JSON serialization
- ISO 8601 dates in JSON
- Pagination metadata
- Standard HTTP status codes

## Performance Considerations

### Database Optimization

**Current:**
- Django ORM automatic indexes
- Foreign key indexes
- Unique constraint indexes

**Query Optimization:**
- select_related() for foreign keys
- prefetch_related() for many-to-many
- Database-level aggregation

### Caching

**Current:** None (not needed at current scale)

**Potential:**
- Redis for session storage
- Database query caching
- Template fragment caching
- CDN for static files

### Static Files

**Current:**
- Django development server
- Local file serving

**Production Options:**
- Nginx static file serving
- Whitenoise middleware
- CDN for global distribution

## Technology Versions Summary

```
Backend Framework:
├── Django 5.2.6
├── djangorestframework 3.16.0
└── Python 3.x

Database:
├── PostgreSQL 15
└── psycopg2-binary 2.9.10

Authentication:
├── django-auth-ldap 4.8.0
└── python-ldap 3.4.4

Supporting Libraries:
├── django-simple-history 3.7.0
├── django-extensions 4.1
├── django-filter 24.2
├── django-cors-headers 4.4.0
├── python-decouple 3.8
├── dj-database-url 2.2.0
└── python-dateutil 2.9.0

Infrastructure:
├── Docker & Docker Compose
├── Nginx Proxy Manager
└── Let's Encrypt SSL
```

## Technology Selection Rationale

### Django Over Flask/FastAPI

**Chosen:** Django
**Reason:**
- Batteries-included (admin, ORM, auth)
- Faster development for database-heavy app
- Built-in security features
- Less third-party dependency management

### PostgreSQL Over MySQL/SQLite

**Chosen:** PostgreSQL
**Reason:**
- Strong data integrity constraints
- Better Django support
- JSON field support for future features
- ACID compliance
- Industry standard for Python apps

### Docker Over Direct Installation

**Chosen:** Docker
**Reason:**
- Reproducible environments
- Easy multi-service orchestration
- Service isolation
- Simplified deployment
- Consistent dev/prod parity

### Server-Side Rendering Over SPA

**Chosen:** Django Templates (SSR)
**Reason:**
- Simpler architecture for solo developer
- No build tooling required
- SEO-friendly
- Fast time-to-interactive
- Progressive enhancement

### Custom LDAP Over django-auth-ldap Backend

**Chosen:** Custom auth handler
**Reason:**
- Specific authentication flow requirements
- Role selection workflow
- Session-based authentication
- Easier debugging and customization
- Greater control over user creation

## Future Technology Considerations

### Potential Additions

**Caching Layer:**
- Redis for session storage
- Caching database queries
- Template fragment caching

**Task Queue:**
- Celery for background jobs
- Email notifications
- Report generation
- Data imports

**Monitoring:**
- Sentry for error tracking
- Prometheus for metrics
- Grafana for dashboards
- Application Performance Monitoring (APM)

**Frontend Enhancement:**
- HTMX for dynamic updates (without full SPA)
- Alpine.js for lightweight interactivity
- Chart.js for visualizations

### Production Upgrades

**WSGI Server:**
- Gunicorn with multiple workers
- uWSGI as alternative
- Process manager (systemd/supervisor)

**Web Server:**
- Nginx for static file serving
- Load balancer for multiple Django instances
- Connection pooling (pgbouncer)

**Security:**
- WAF (Web Application Firewall)
- Rate limiting
- DDoS protection
- Security headers (HSTS, CSP)

## Conclusion

Resupekka's technology stack is deliberately chosen for:
- **Stability** - Mature, well-tested technologies
- **Maintainability** - Django's conventions and documentation
- **Integration** - LDAP, PostgreSQL, Docker ecosystem
- **Simplicity** - Minimal dependencies, no over-engineering
- **Solo Development** - Batteries-included framework reduces decisions

The stack successfully serves 582 users across 5 departments in a live production environment, demonstrating that thoughtful technology choices can deliver enterprise-grade systems even in first-time solo development scenarios.