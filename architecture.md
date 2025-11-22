---

# **Architecture Document (Architecture.md)**

**Product:** Hotel Calendar & Room Pricing Management System
**Version:** 1.0
**Status:** Draft
**Owner:** Engineering Team

---

# **1. System Overview**

The system enables hotel administrators to manage **room availability** and **prices** on a daily calendar.

It exposes REST APIs to be consumed by:

* Admin dashboard (internal)
* Booking engine / website (external)
* Channel managers (future extension)

Core responsibilities:

1. CRUD on daily room inventory
2. CRUD on daily room pricing
3. Bulk operations
4. Validation against hotel capacity
5. Audit logging
6. Fast read APIs for booking engines

---

# **2. High-Level Architecture**

```
 ┌───────────────────────────────┐
 │        Admin Frontend         │
 │  (React / Next.js Web UI)     │
 └───────────────┬───────────────┘
                 │
                 ▼
 ┌───────────────────────────────┐
 │     API Gateway / Backend     │
 │ Node.js / Spring Boot / Django│
 │  Auth, Business Logic, Validation │
 │  Pricing Rules, Audit Logging │
 └───────────────┬───────────────┘
                 │
                 ▼
       ┌───────────────────────┐
       │     Database Layer    │
       │  PostgreSQL / MySQL   │
       │  hotel_calendar       │
       │  audit_logs           │
       │  room_types           │
       └───────────────────────┘
                 │
                 ▼
      ┌──────────────────────────┐
      │   Caching Layer (Redis)  │
      │   Fast reads for booking │
      │   Cached calendars       │
      └──────────────────────────┘
                 │
                 ▼
      ┌──────────────────────────┐
      │  External Consumers      │
      │  (Booking Engines)       │
      │  GET availability, price │
      └──────────────────────────┘
```

---

# **3. Backend Architecture Details**

### **3.1 Service Layers**

#### **1. API Layer**

* Handles routing
* JWT authentication
* Request validation

Example:
`POST /admin/calendar/update`
`GET /availability`

#### **2. Business Logic Layer**

* Capacity validation
* Prevent overbooking
* Merge date-range updates
* Price normalization
* Audit logging

#### **3. Data Access Layer**

* ORM (Hibernate / Django ORM / Prisma)
* Transaction-safe writes

#### **4. Cache Layer**

* Redis stores:

  * Calendar month view
  * Room availability map
* TTL 5 minutes
* Auto invalidated on update

---

# **4. Database Architecture**

### **Tables**

1. **rooms_master**
   Stores total room capacity of each type.

2. **hotel_calendar**
   Stores availability and price for each date.

3. **audit_logs**
   Stores every data modification.

4. **users**
   Admin login / RBAC.

5. **bulk_update_jobs**
   For async bulk jobs.

---

# **5. API Architecture**

### **Read APIs (public for Booking Engine)**

* Highly optimized
* Served via Redis cache
* Idempotent, low latency
* Response < 200 ms

### **Write APIs (admin only)**

* Always update DB
* Invalidate/calibrate cache
* Generate audit logs

---

# **6. Module Breakdown**

### **1. Calendar Module**

* CRUD for daily entries
* Month-level fetch
* Day-name mapping

### **2. Pricing Module**

* CRUD on daily prices
* Price consistency checks

### **3. Bulk Edit Module**

* Apply updates to date ranges
* Async job queue (optional)

### **4. Availability Module**

* Check if N rooms exist in date range

### **5. Authentication & RBAC Module**

* JWT
* Role-based controls: admin, viewer

### **6. Audit Module**

* Store before/after snapshot

---

# **7. Caching Strategy**

### **What is cached**

* Monthly calendar
* Availability results
* Pricing range results

### **Cache Invalidation**

Any update → invalidate that month’s cache key:

```
calendar:{yyyy-mm}
availability:{yyyy-mm-dd}:{room_type}
```

### **Why Redis**

* Fast read
* Must support 10–15 booking engines hitting APIs per second

---

# **8. Scalability Considerations**

### **Horizontal Scaling**

* Backend is stateless → can run multiple replicas
* Redis shared cache
* DB read replicas (future)

### **Partitioning Strategy (future)**

* Partition hotel_calendar by year
* Sharding by property_id (for multi-hotel support)

---

# **9. Security Architecture**

1. JWT token required for all admin APIs
2. Password hashing (bcrypt/argon2)
3. Rate limiting booking APIs
4. CORS controlled
5. Input sanitization

---

# **10. Deployment Architecture**

### **Environment**

* Dev
* QA
* Prod

### **Infra**

* Docker + Kubernetes
* Nginx reverse proxy
* HTTPS enforced

### **Logging**

* JSON logs
* Stored in ELK/Grafana
* Audit logs in DB

---

# **11. Monitoring Metrics**

| Layer          | Metrics                      |
| -------------- | ---------------------------- |
| API Layer      | RPS, latency, 4xx/5xx errors |
| DB Layer       | slow queries, locks          |
| Cache Layer    | hit ratio                    |
| Business Layer | update frequency             |
| Infra          | CPU, RAM, pod restarts       |

---

# **12. Risks & Mitigations**

| Risk                          | Impact | Mitigation               |
| ----------------------------- | ------ | ------------------------ |
| Incorrect price updates       | High   | Audit logs + rollback    |
| Cache stale reads             | Medium | Short TTL + invalidation |
| Overbooking                   | High   | Capacity validation      |
| Slow read during peak         | Medium | Redis caching            |
| Future multi-hotel complexity | High   | Add property_id now      |

---

# **13. Future Extensions**

* OTA sync (Booking.com, MMT)
* Dynamic pricing rules
* Multi-room-type support
* Multi-property support
* Seasonal rate templates

---
