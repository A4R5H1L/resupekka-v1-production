# Database Schema

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           USER & AUTHENTICATION                          │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│      users_user          │  (Custom User Model extending AbstractUser)
├──────────────────────────┤
│ PK: id (BigInt)          │
│    email (unique)        │  ← USERNAME_FIELD (authentication)
│    first_name            │  ← From LDAP
│    last_name             │  ← From LDAP
│    nickname (unique)     │
│    department            │  ← From LDAP
│    company               │  ← From LDAP
│    manager_username      │  ← From LDAP
│    manager_email         │  ← From LDAP
│    preferred_research_un │  (for dashboard filtering)
│    is_staff              │  (Django admin access)
│    is_active             │
│    is_superuser          │
│    date_joined           │
│    last_login            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by:
           ├──────────────────────────────────────────┐
           │                                          │
           │                                          │

┌──────────────────────────────────────────────────────────────────────────┐
│                         CORE ENTITIES                                     │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│       core_role          │  (System roles: User, Manager, Admin, Director)
├──────────────────────────┤
│ PK: id                   │
│    name (unique)         │  ← "User", "Manager", "Admin", "Director"
│    description           │
│    created_at            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by: Contract
           │

┌──────────────────────────┐
│   core_researchunit      │  (Research units: TKI, RoboAI, etc.)
├──────────────────────────┤
│ PK: id                   │
│    name (unique)         │  ← "TKI", "RoboAI tutkimuskeskus", "Unknown"
│    description           │
│ FK: manager_id           │  → allocations_contract
│    created_at            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by: Project, Contract
           │

┌──────────────────────────┐
│   core_projectstatus     │  (Project lifecycle states)
├──────────────────────────┤
│ PK: id                   │
│    name (unique)         │  ← "Active", "Finished", "BeingPrepared", etc.
│    description           │
│    created_at            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by: Project
           │

┌──────────────────────────┐
│      core_project        │  (Research projects)
├──────────────────────────┤
│ PK: id                   │
│    name (unique)         │
│    short_name (unique)   │
│    description           │
│ FK: unit_id              │  → core_researchunit
│ FK: status_id            │  → core_projectstatus
│    start_date            │
│    end_date              │
│    end_date_original     │  (track extensions)
│    website               │
│    notification_email    │  (comma-separated emails)
│ FK: created_by_id        │  → users_user
│    created_at            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by: WorkAllocation
           │ Historical: historicalproject (django-simple-history)


┌──────────────────────────────────────────────────────────────────────────┐
│                      ALLOCATION ENTITIES                                  │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│ allocations_contract     │  (Personnel contracts)
├──────────────────────────┤
│ PK: id                   │
│ FK: user_id              │  → users_user
│ FK: role_id              │  → core_role
│ FK: unit_id              │  → core_researchunit
│    start_date            │
│    end_date              │
│    work_percentage       │  (0-100%, default 100)
│    note                  │
│    created_at            │
│    updated_at            │
├──────────────────────────┤
│ UNIQUE: (user, role,     │  ← Prevents duplicate contracts
│         unit, start_date,│
│         end_date)        │
└──────────┬───────────────┘
           │
           │ Referenced by: WorkAllocation, ResearchUnit.manager
           │

┌──────────────────────────┐
│allocations_workallocation│  (Monthly work allocation types)
│         type             │
├──────────────────────────┤
│ PK: id                   │
│    name (unique)         │  ← "Normal", "Flat Rate"
│    description           │
│    created_at            │
│    updated_at            │
└──────────┬───────────────┘
           │
           │ Referenced by: WorkAllocation
           │

┌──────────────────────────┐
│allocations_workallocation│  (Monthly work allocations)
├──────────────────────────┤
│ PK: id                   │
│ FK: user_id              │  → users_user
│ FK: contract_id          │  → allocations_contract
│ FK: project_id           │  → core_project
│ FK: allocation_type_id   │  → allocations_workallocationtype
│    month                 │  (First day of month)
│    allocation_percentage │  (0-100%)
│    note                  │
│    created_at            │
│    updated_at            │
├──────────────────────────┤
│ UNIQUE: (user, project,  │  ← Prevents duplicate monthly allocations
│         allocation_type, │
│         month)           │
└──────────┬───────────────┘
           │
           │ Referenced by: WorkAllocationRequest
           │ Historical: historicalworkallocation (django-simple-history)


┌──────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW ENTITIES                                  │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│workflows_workallocation  │  (Approval workflow for allocation changes)
│        request           │
├──────────────────────────┤
│ PK: id                   │
│ FK: allocation_id        │  → allocations_workallocation
│    allocation_percentage │  (Requested new percentage)
│    original_percentage   │  (Previous percentage)
│    note                  │
│    access_token          │  (Token-based approval link)
│    status                │  ← "pending", "approved", "rejected"
│ FK: created_by_id        │  → users_user (requester)
│ FK: approved_by_id       │  → users_user (approver)
│    approved_at           │
│    created_at            │
│    updated_at            │
└──────────────────────────┘
           │ Historical: historicalworkallocationrequest


┌──────────────────────────────────────────────────────────────────────────┐
│                       AUDIT TRAIL (Simple History)                        │
└──────────────────────────────────────────────────────────────────────────┘

For each tracked model, django-simple-history creates:

┌──────────────────────────┐
│ historical<modelname>    │  (e.g., historicaluser, historicalproject)
├──────────────────────────┤
│ PK: history_id           │
│    <all model fields>    │  (snapshot of data)
│    history_date          │  (when change occurred)
│    history_change_reason │
│    history_type          │  ("+": created, "~": updated, "-": deleted)
│ FK: history_user_id      │  → users_user (who made the change)
└──────────────────────────┘

Tracked Models:
  • users_user → historicaluser
  • core_project → historicalproject
  • allocations_workallocation → historicalworkallocation
  • workflows_workallocationrequest → historicalworkallocationrequest
```

## Entity Relationships

### User Relationships

```
users_user (1) ───────────── (M) allocations_contract
    │                                    │
    │                                    │
    │ (1) ─────────────────────── (M) allocations_workallocation
    │                                    │
    │ (1) ─────────────────────── (M) workflows_workallocationrequest (created_by)
    │ (1) ─────────────────────── (M) workflows_workallocationrequest (approved_by)
    │ (1) ─────────────────────── (M) core_project (created_by)
```

### Project Relationships

```
core_researchunit (1) ─────── (M) core_project
core_projectstatus (1) ───────── (M) core_project
core_project (1) ─────────────── (M) allocations_workallocation
```

### Contract & Allocation Flow

```
users_user (1) ────┐
                   ├──── (M) allocations_contract ────┐
core_role (1) ─────┤                                  │
                   │                                  │
core_researchunit (1) ──┘                            │
                                                      │
                                                      │ (1)
                                                      │
allocations_workallocationtype (1) ────┐             │
                                       ├─ (M) allocations_workallocation
core_project (1) ──────────────────────┤             │
                                       │             │
users_user (1) ────────────────────────┘             │
                                                      │ (1)
                                                      │
                                                (M) workflows_workallocationrequest
```

## Key Concepts

### 1. User Model

**Custom user extending AbstractUser:**
- Email-based authentication (no username field)
- LDAP-synchronized attributes
- Role not stored in database (session-based)
- Manager preferences for dashboard filtering

**Key Fields:**
- `email` - Unique identifier and login credential
- `nickname` - Display name (unique)
- `department`, `company` - LDAP organizational info
- `manager_username`, `manager_email` - LDAP manager hierarchy
- `preferred_research_unit` - Dashboard filtering preference

### 2. Role System

**Four system roles:**
- **User** - Basic access, view own allocations
- **Manager** - Manage team allocations
- **Admin** - Cross-unit administrative access
- **Director** - Full system access

**Note:** Roles stored in session, not as user field
- Allows same user to have different roles
- Role selection happens after login
- Flexible permission model

### 3. Research Units

**Organizational structure:**
- TKI (Research, Development, and Innovation)
- RoboAI Research Center
- Unknown (fallback)

**Purpose:**
- Group projects by research area
- Manage personnel within units
- Dashboard filtering

**Manager Field:**
- Links to a Contract (not directly to User)
- Manager is a user with a contract in that unit
- Allows tracking manager role over time

### 4. Projects

**Project lifecycle:**
- Being Prepared → Submitted → Active → Finished/Rejected

**Key Dates:**
- `start_date`, `end_date` - Current project timeline
- `end_date_original` - Original planned end date (track extensions)

**Notifications:**
- `notification_email` - Comma-separated list
- Project manager, secretary, external collaborators

### 5. Contracts

**Personnel contract linking:**
- User + Role + Research Unit
- Date range (start_date to end_date)
- Work percentage (e.g., 80% if part-time)

**Foundation for allocations:**
- Must have a contract to have work allocations
- Contract defines user's relationship to unit
- Work percentage is total capacity

### 6. Work Allocations

**Monthly allocation tracking:**
- First day of month (e.g., 2025-01-01 for January)
- Percentage of work time (0-100%)
- Linked to contract and project

**Allocation Types:**
- **Normal** - Regular work allocation
- **Flat Rate** - Fixed-rate work

**Business Rule:**
- Sum of allocations for a user in a month should not exceed contract work_percentage
- Example: 100% contract → max 100% total allocations per month

### 7. Workflow Requests

**Approval workflow:**
- User requests allocation change
- System generates access token
- Email sent to approver with token link
- Approver clicks link to approve/reject
- Audit trail tracks approval

**Fields:**
- `allocation_percentage` - Requested new value
- `original_percentage` - Previous value (for audit)
- `access_token` - Unique token for approval link
- `status` - pending, approved, rejected

### 8. Audit Trail (Simple History)

**Change tracking:**
- Every change creates historical record
- Tracks: who, what, when
- Immutable audit log
- Accessible via model.history.all()

**Use Cases:**
- Compliance reporting
- Change audits
- Rollback information
- Dispute resolution

## Database Constraints

### Unique Constraints

**Contract:**
```sql
UNIQUE (user_id, role_id, unit_id, start_date, end_date)
```
- Prevents duplicate contracts
- Same user can have multiple contracts with different roles/units/dates

**WorkAllocation:**
```sql
UNIQUE (user_id, project_id, allocation_type_id, month)
```
- Prevents duplicate monthly allocations
- One allocation per user-project-type-month combination

**User:**
```sql
UNIQUE (email)
UNIQUE (nickname)
```
- Email is primary identifier
- Nickname must be unique for display

**Project:**
```sql
UNIQUE (name)
UNIQUE (short_name)
```
- Project names must be unique
- Short names must be unique

### Foreign Key Constraints

**All foreign keys use ON DELETE RESTRICT:**
- Prevents accidental deletion of referenced data
- Must explicitly handle dependencies before deletion
- Data integrity protection

**Example:**
```python
user = models.ForeignKey('users.User', on_delete=models.RESTRICT)
```
- Cannot delete a User if they have Contracts
- Cannot delete a Project if it has WorkAllocations
- Must clean up dependencies first

### Validation Constraints

**Percentage Fields:**
```python
allocation_percentage = models.FloatField(
    validators=[MinValueValidator(0), MaxValueValidator(100)]
)
```
- Work percentages: 0-100%
- Enforced at database and application level

## Sample Data Relationships

### Example: User with Multiple Allocations

```
User: johanna.virtanen@samk.fi
  │
  ├── Contract 1:
  │     Role: Researcher
  │     Unit: TKI
  │     Period: 2024-01-01 to 2025-12-31
  │     Work %: 100%
  │     │
  │     ├── WorkAllocation (Jan 2025):
  │     │     Project: AI Research Project
  │     │     Type: Normal
  │     │     Allocation: 50%
  │     │
  │     └── WorkAllocation (Jan 2025):
  │           Project: Robotics Initiative
  │           Type: Normal
  │           Allocation: 30%
  │
  └── Historical Records:
        ├── Created 2024-01-15 by admin@samk.fi
        ├── Updated 2024-03-20 by manager@samk.fi (department change)
        └── Updated 2025-01-10 by johanna.virtanen@samk.fi (preferred unit)
```

### Example: Project with Team

```
Project: "AI-Powered Manufacturing Optimization"
  Short Name: "AI-MFG-2025"
  Unit: TKI
  Status: Active
  Period: 2024-06-01 to 2026-05-31
  Created By: project.manager@samk.fi
  │
  ├── Team Member 1:
  │     User: researcher1@samk.fi
  │     WorkAllocation: 40% (Normal, Jan 2025)
  │
  ├── Team Member 2:
  │     User: researcher2@samk.fi
  │     WorkAllocation: 60% (Normal, Jan 2025)
  │
  └── Team Member 3:
        User: consultant@samk.fi
        WorkAllocation: 20% (Flat Rate, Jan 2025)
```

### Example: Approval Workflow

```
WorkAllocationRequest:
  Allocation: researcher1@samk.fi → AI-MFG-2025 (Jan 2025)
  Original: 40%
  Requested: 60%
  Created By: researcher1@samk.fi
  Status: pending
  Access Token: "abc123def456..."
  │
  Email sent to: project.manager@samk.fi
  Link: https://resupekka.samk.fi/dashboard/approve-request/?token=abc123def456
  │
  ↓ (Manager clicks link)
  │
  Status: approved
  Approved By: project.manager@samk.fi
  Approved At: 2025-01-15 10:30:00
  │
  ↓ (System updates allocation)
  │
  WorkAllocation updated: 40% → 60%
  HistoricalWorkAllocation created (audit trail)
```

## Database Queries

### Common Queries

**Get user's allocations for current month:**
```python
from datetime import datetime
from apps.allocations.models import WorkAllocation

current_month = datetime.now().replace(day=1)
allocations = WorkAllocation.objects.filter(
    user=user,
    month=current_month
).select_related('project', 'contract', 'allocation_type')
```

**Get all projects for a research unit:**
```python
from apps.core.models import Project

projects = Project.objects.filter(
    unit__name='TKI',
    status__name='Active'
).select_related('unit', 'status', 'created_by')
```

**Check user's total allocation for a month:**
```python
from django.db.models import Sum

total_allocation = WorkAllocation.objects.filter(
    user=user,
    month=current_month
).aggregate(total=Sum('allocation_percentage'))['total'] or 0

# Should not exceed contract.work_percentage
```

**Get project team members:**
```python
from apps.allocations.models import WorkAllocation

team = WorkAllocation.objects.filter(
    project=project,
    month=current_month
).select_related('user', 'contract').distinct('user')
```

**Get allocation change history:**
```python
allocation = WorkAllocation.objects.get(id=allocation_id)
history = allocation.history.all().order_by('-history_date')

for record in history:
    print(f"{record.history_date}: {record.allocation_percentage}% by {record.history_user}")
```

## Performance Considerations

### Indexes

**Automatic Django indexes:**
- Primary keys (all tables)
- Foreign keys (automatic index)
- Unique fields (email, nickname, project name)

**Query Optimization:**
- Use `select_related()` for foreign keys (one-to-one, many-to-one)
- Use `prefetch_related()` for reverse foreign keys (one-to-many)
- Filter on indexed fields when possible

### Historical Tables Growth

**django-simple-history creates records on every change:**
- Can grow large over time
- Consider archiving old historical records
- Indexes on history_date for performance

## Backup and Recovery

### Critical Tables

**Priority 1 (Data loss catastrophic):**
- users_user
- core_project
- allocations_contract
- allocations_workallocation

**Priority 2 (Data loss significant):**
- workflows_workallocationrequest
- core_role
- core_researchunit
- core_projectstatus
- allocations_workallocationtype

**Priority 3 (Can be regenerated):**
- historical* tables (audit trail - important but can be reconstructed from backups)

### Backup Strategy

**Recommended:**
```bash
# Daily full backup
pg_dump -U resupekka_user resupekka_db > backup_$(date +%Y%m%d).sql

# Retention: 30 days
# Off-site storage for disaster recovery
```

## Schema Evolution

### Migration History

**Initial migration:**
- User model with LDAP fields
- Core models (Project, Role, ResearchUnit)
- Allocation models (Contract, WorkAllocation)
- Workflow models (WorkAllocationRequest)
- SimpleHistory integration

**Future migrations may include:**
- Additional fields for new features
- New models for expanded functionality
- Index optimizations
- Data migrations for cleanup

### Versioning

Django migrations track schema changes:
- Each app has migrations/ directory
- Sequential migration files
- Applied via `python manage.py migrate`
- Rollback possible with `python manage.py migrate <app> <migration_name>`

## Conclusion

Resupekka's database schema is designed for:

**Data Integrity:**
- Foreign key constraints
- Unique constraints
- Validation at model level

**Auditability:**
- django-simple-history on critical models
- Created/updated timestamps
- User tracking for changes

**Flexibility:**
- Session-based roles (not rigid user role field)
- Multiple contracts per user
- Multiple allocations per user per project

**Performance:**
- Indexed foreign keys
- Efficient query patterns
- Select/prefetch related for optimization

The schema successfully supports 582 users, 5,948 allocations, and 57 projects across 9 research units with proven scalability.