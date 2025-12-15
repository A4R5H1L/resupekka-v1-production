# Resupekka V1 - Enterprise Resource Management System

> Live production system serving university research operations at SAMK

![Django](https://img.shields.io/badge/Django-5.2.6-092E20?style=flat&logo=django)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat&logo=python)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-316192?style=flat&logo=postgresql)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker)
![Status](https://img.shields.io/badge/Status-Production-success)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat)

## Overview

Resupekka is a comprehensive work allocation and project management system designed for academic research operations. The system tracks personnel contracts, project assignments, and work allocations across multiple research units at Satakunta University of Applied Sciences (SAMK).

**Current Status:** Live production system serving 582 users across 5 departments
**Production URL:** `https://resupekka.samk.fi`
**Deployment Date:** August 2025
**Uptime:** 4+ months of stable operation

## Production Metrics

Current system scale (as of December 2025):

- **582 Registered Users** - Synchronized from SAMK LDAP directory
- **5,948 Work Allocations** - Tracked across 57 research projects
- **57 Active Projects** - Spanning 2024-2029 (5-year planning horizon)
- **9 Research Units** - RoboAI, RoboAI Health, WANDER, Tourism, Marine Logistics, and more
- **171 Active Contracts** - Current personnel assignments
- **261 Workflow Approvals** - Processed allocation change requests

### Real System Activity (Last 90 Days)

- **3,904 allocations modified** - Active resource management
- **33 projects created** - Continuous project pipeline
- **177 approval requests processed** - Workflow approval system actively used
- **67 users actively managing allocations** - Managers and project leads
- **507 allocations modified in last 30 days** - Daily active usage

### Department Coverage

- **TKI** (Research, Development & Innovation): 146 users
- **PATA** (Services): 98 users
- **HYVO** (Wellbeing): 97 users
- **TECH** (Technology): 85 users
- **OPVA** (Educational Services): 62 users

## Development Journey

### First Software Engineering Project

Resupekka was developed as my **first professional software project** during a 4-month internship at SAMK:

**Challenge:** University research units were managing work allocations across 500+ staff members using Excel spreadsheets. No centralized system, no approval workflows, no audit trails, and manual effort to track resource allocation across dozens of projects.

**Solution:** Built a full-stack Django application from scratch with:
- Enterprise LDAP integration for Single Sign-On
- Role-based access control with 4-tier permission system
- Approval workflow system with email notifications
- Complete audit trail using django-simple-history
- Docker-based deployment with automated SSL

**Timeline:**
- **Month 1:** Requirements gathering, technology selection, database design
- **Month 2:** Core development (Django models, LDAP auth, basic CRUD)
- **Month 3:** Advanced features (approval workflows, dashboards, REST API)
- **Month 4:** Deployment, data migration (582 users, initial allocations), user training

**Outcome:**
- Deployed to production August 2025
- 4+ months of stable operation with zero downtime
- 582 users actively using the system
- 177 approval workflows processed in last 90 days
- Successfully replaced manual Excel-based processes

**Learning Curve:**
- First time using Django framework
- First time deploying production infrastructure (Docker, Nginx, SSL)
- First time integrating with enterprise LDAP (Active Directory)
- First time designing a database schema for real users
- First time managing production deployment solo

## Skills Demonstrated

### Full-Stack Development
- **Backend:** Django 5.2.6 with Django REST Framework, custom authentication handlers
- **Database:** PostgreSQL schema design with foreign key constraints, audit tables, complex queries
- **Frontend:** Django templates, server-side rendering, responsive dashboards
- **API Design:** RESTful endpoints with token authentication, pagination, filtering

### DevOps & Infrastructure
- **Containerization:** Docker Compose orchestration for multi-service application
- **Reverse Proxy:** Nginx Proxy Manager configuration with SSL termination
- **SSL/TLS:** Let's Encrypt automated certificate management
- **Network Security:** Docker network isolation, firewall rules, port management
- **Deployment:** Production deployment on Ubuntu server with SSH access
- **Monitoring:** Container health checks, database monitoring, log management

### Enterprise Integration
- **LDAP/Active Directory:** Custom authentication against SAMK LDAP server (ad-ldap.samk.fi)
- **Single Sign-On:** Session-based authentication with university credentials
- **User Synchronization:** Automatic user creation/update from LDAP attributes
- **Department Hierarchy:** Manager relationship mapping from Active Directory

### System Design & Architecture
- **Three-tier architecture:** Web (Nginx) → Application (Django) → Database (PostgreSQL)
- **Security hardening:** CSRF protection, SQL injection prevention, XSS protection
- **Audit trails:** django-simple-history for compliance and change tracking
- **Workflow automation:** Token-based approval system with email notifications
- **Data modeling:** Complex relationships (Users, Contracts, Allocations, Projects, Approvals)

### Database & Data Management
- **Schema Design:** 10+ interconnected tables with proper constraints
- **Data Management:** 582 users synchronized from LDAP, 5,948 allocations created via app workflows
- **Query Optimization:** select_related(), prefetch_related() for performance
- **Data Integrity:** Foreign key constraints, unique constraints, validation rules
- **Historical Tracking:** Automatic audit logging for all data changes

### Problem Solving
- **LDAP Integration:** Built custom authentication handler (not using django-auth-ldap backend) for greater control
- **Approval Workflow:** Designed token-based system to allow approvals via email links
- **Role Management:** Session-based roles (not database field) for flexible permission model
- **Network Isolation:** Configured Docker networks to prevent unauthorized access
- **CSRF with Proxy:** Solved CSRF validation issues when running behind reverse proxy

## Tech Stack

### Backend Framework
- **Django 5.2.6** - Web framework with built-in admin, ORM, authentication
- **Django REST Framework 3.16.0** - API layer with serializers and viewsets
- **PostgreSQL 15** - Relational database with ACID compliance
- **psycopg2-binary 2.9.10** - PostgreSQL database adapter

### Authentication & Security
- **django-auth-ldap 4.8.0** - LDAP integration library
- **python-ldap 3.4.4** - LDAP protocol client
- **SAMK LDAP Server** - Active Directory integration (`ad-ldap.samk.fi`)

### Infrastructure
- **Docker Compose** - Multi-container orchestration
- **Nginx Proxy Manager** - Reverse proxy with web UI management
- **Let's Encrypt** - Automated SSL certificate generation and renewal
- **Ubuntu Server** - Production Linux environment

### Supporting Libraries
- **django-simple-history 3.7.0** - Automatic audit trail and change tracking
- **django-extensions 4.1** - Enhanced management commands
- **django-filter 24.2** - Dynamic query filtering for API
- **django-cors-headers 4.4.0** - Cross-origin resource sharing
- **python-decouple 3.8** - Environment variable management
- **python-dateutil 2.9.0** - Advanced date/time operations

## Key Features

1. **LDAP Authentication** - Single Sign-On with SAMK Active Directory (582 users)
2. **Role-Based Access Control** - 4-tier system (User, Manager, Admin, Director)
3. **Project Management** - 57 projects across 9 research units with team tracking
4. **Work Allocation Tracking** - 5,948 monthly percentage-based allocations
5. **Contract Management** - 171 active personnel contracts with date ranges
6. **Approval Workflows** - 261 processed requests with token-based email approval
7. **Audit Dashboard** - Complete change history with django-simple-history
8. **Multi-Department Support** - 5 departments (TKI, PATA, HYVO, TECH, OPVA)
9. **Research Unit Organization** - 9 research centers and units
10. **REST API** - Full programmatic access with token authentication
11. **Dashboard Analytics** - Real-time allocation visibility for managers
12. **Admin Interface** - Django admin for system management

## Architecture Overview

```
User Browser → Nginx Proxy Manager (SSL) → Django Application → PostgreSQL Database
                                         ↘ LDAP Server (Authentication)
```

**Network Security:**
- Public: Only ports 80/443 (HTTPS) exposed
- Internal: Django (8000), PostgreSQL (5432), MCP (8001) network-isolated
- Admin: PgAdmin on localhost:8080, Nginx admin on VPN-only port 81

For detailed architecture diagrams and system design:
- [System Architecture Diagram](diagrams/system-architecture.md) - Request flow, component interaction
- [Database Schema](diagrams/database-schema.md) - Entity relationships, data model
- [Architecture Documentation](docs/ARCHITECTURE.md) - Deep dive into system design

## Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System components, middleware stack, Django app structure
- **[DEPLOYMENT.md](docs/DEPLOYMENT.md)** - Docker setup, environment configuration, deployment process
- **[SCALE_AND_IMPACT.md](docs/SCALE_AND_IMPACT.md)** - Business impact, time savings, adoption metrics
- **[TECHNICAL_STACK.md](docs/TECHNICAL_STACK.md)** - Technology choices, library versions, rationale

## Deployment Architecture

Production environment using Docker Compose:

1. **django-api** - Django application server (port 8000, internal)
2. **db** - PostgreSQL 15 database (port 5432, internal)
3. **mcp-server** - Model Context Protocol support service (port 8001, internal)
4. **pgadmin** - Database administration UI (localhost:8080, admin only)
5. **Nginx Proxy Manager** - Public HTTPS gateway with SSL termination

**Infrastructure Features:**
- Network isolation via Docker bridge networks
- Automated SSL certificate renewal
- Persistent data volumes for PostgreSQL
- Health checks for database availability
- Restart policies for high availability

## Security Implementation

- **SSL/TLS Encryption** - Let's Encrypt certificates with automatic renewal
- **LDAP Integration** - No password storage in database, centralized authentication
- **CSRF Protection** - Django tokens with reverse proxy header configuration
- **Network Isolation** - Docker networks prevent direct access to services
- **Session Management** - 4-hour timeout with custom middleware
- **Audit Logging** - django-simple-history tracks all data modifications
- **Role-Based Permissions** - View-level authorization checks
- **SQL Injection Prevention** - Django ORM parameterized queries
- **XSS Protection** - Template auto-escaping

## Business Impact

### Problem Solved
**Before Resupekka:** Work allocations managed in Excel spreadsheets across multiple departments. Manual effort to compile reports, no approval workflow, no audit trail, difficult to track resource conflicts or over-allocation.

**After Resupekka:**
- **Centralized Data:** Single source of truth for 5,948 allocations
- **Approval Workflow:** 261 requests processed with email-based approval system
- **Audit Compliance:** Complete change history for all modifications
- **Resource Visibility:** Managers can see team allocations in real-time
- **Time Savings:** ~350 hours/year saved in administrative overhead
- **Data Integrity:** Database constraints prevent duplicate or invalid allocations

### Active Usage Statistics
- **507 allocations modified** in last 30 days (daily active management)
- **261 approval requests** processed through app workflow (no manual emails or messaging needed)
- **6 new projects** created in last 30 days (continuous project pipeline)
- **67 users** actively managing allocations (managers, admins)

### Departments Served
- Research units (TKI, RoboAI, WANDER, Tourism, Marine Logistics, etc.)
- University administration (strategic planning, compliance)
- Project managers (team allocation, resource planning)
- HR coordination (contract tracking, department assignments)

## Future Roadmap

**Planned Expansions:**
- Expand to teaching staff allocation (additional 300+ users)
- HR integration for automated contract management
- Advanced analytics and reporting dashboards
- Integration with financial systems for budget tracking
- Mobile-responsive UI improvements
- Replace legacy Reportronic system university-wide

## Getting Real-Time Metrics

Connect to production database to query current metrics:

```bash
# Total users
docker-compose exec django-api python manage.py shell -c \
  "from apps.users.models import User; print(f'Users: {User.objects.count()}')"

# Total allocations
docker-compose exec django-api python manage.py shell -c \
  "from apps.allocations.models import WorkAllocation; print(f'Allocations: {WorkAllocation.objects.count()}')"

# Active projects
docker-compose exec django-api python manage.py shell -c \
  "from apps.core.models import Project; print(f'Projects: {Project.objects.count()}')"

# Recent activity (last 30 days)
docker-compose exec django-api python manage.py shell -c \
  "from apps.allocations.models import WorkAllocation; from datetime import timedelta; from django.utils import timezone; \
   recent = WorkAllocation.objects.filter(updated_at__gte=timezone.now() - timedelta(days=30)).count(); \
   print(f'Modified (30d): {recent}')"
```

## Lessons Learned

**Technical Insights:**
- Django's batteries-included approach perfect for rapid development
- Custom LDAP handler provided more flexibility than library backends
- Docker Compose adequate for single-server deployment at this scale
- Network isolation crucial even in internal university environment
- Audit trails (django-simple-history) invaluable for compliance

**Project Management:**
- Started with research units (pilot users) before expanding
- Iterative feedback from managers improved UX significantly
- Real production usage created organic data growth (57 projects, 5,948 allocations via workflows)
- User training essential for adoption

**What Worked Well:**
- Django admin interface saved weeks of development time
- LDAP integration eliminated password management complexity
- Token-based approval workflow simple yet effective
- Docker deployment reproducible and maintainable

## Acknowledgments

**Note:** Source code is proprietary to SAMK (Satakunta University of Applied Sciences). This repository contains architecture documentation only.

**Developed during:** Software Engineering Internship at SAMK (4 months)
**Role:** Solo Developer (full-stack, DevOps, deployment)
**Technologies:** Django, PostgreSQL, Docker, Nginx, LDAP

**Deployed:** August 2025
**Status:** Active production system serving 582 users
**Uptime:** 4+ months with zero downtime

> **Note:** Production system is network-restricted (accessible only within SAMK infrastructure)
