# 1️⃣ Big Picture: What you are actually building

Strip away the business language and this is the real system:

> A role-based system where Admins create QR sessions and Users scan them, and every scan writes a signed attendance record with anti-fraud protection and analytics.

At its heart, SAS is:

```
AUTH → QR SESSION → SCAN → VALIDATE → STORE → REPORT
```

---

# 2️⃣ Architecture (Golden Standard Version)

### Logical Architecture

```
Flutter App (Scanner)
        |
        v
   API Gateway
        |
        |
   Auth Service
   Attendance Service
   Reporting Service
        |
        v
    Database
```

---

### Recommended Stack (For students)

Use one backend stack only:

✅ .NET Web API
✅ PostgreSQL
✅ JWT for auth
✅ Identity Core (optional advanced)

They should *not* find NestJS easier than .NET for this level system.

---

# 3️⃣ Database Design (core tables)

### Users

```
Id (GUID)
Email
FullName
Role (Student / Staff / Admin)
StudentId / StaffId
Department
PhotoUrl
```

---

### AttendanceSessions (QR)

```
Id
Type (Daily / Department / Event)
ExpiresAt
DepartmentId
CreatedBy
IsActive
SecretHash
```

This is your QR code session.

---

### AttendanceLogs

```
Id
UserId
SessionId
Timestamp
DeviceId
Latitude
Longitude
Status (Present / Late)
```

This is the proof log.

---

### AuditLogs

```
Id
AdminId
Action
RecordId
Timestamp
Reason
```

Because your document requires **audit compliance**.

---

# 4️⃣ QR Code Generation Logic (anti-cheat)

### Admin generates QR:

Backend returns:

```
Base64(
   SessionId
   ExpiryTimestamp
   SignatureHash
)
```

**Never embed raw IDs without signing them**.

---

### How QR validation works:

When a student scans:

1. Decode QR
2. Check timestamp not expired
3. Validate signature hash
4. Check user not already scanned
5. Store log

If any fails → reject.

---

### Tutor gold line:

> "QR codes are not images; they are encrypted instructions."

Teach that.

---

# 5️⃣ API Design (Minimal Required Endpoints)

### Authentication

```
POST /api/auth/login
POST /api/auth/register
```

---

### QR

```
POST /api/qr/create
GET  /api/qr/active
```

---

### Attendance

```
POST /api/attendance/scan
GET  /api/attendance/user
GET  /api/attendance/report
```

---

### Admin

```
POST /api/admin/adjust
GET  /api/admin/logs
```

---

# 6️⃣ Business Rules Students MUST implement

From your document :

✅ QR expires automatically
✅ No duplicate scans per session
✅ Admin can override with reason
✅ Attendance status auto-calculated
✅ Audit logs immutable
✅ Roles restrict access
✅ Response API format:

```
status
message
data
```

---

# 7️⃣ Analytics (Simplified student version)

Don’t overcomplicate:

Implement:

✅ Attendance count by day
✅ Attendance per department
✅ Absentees list

Later optional:

* Trend charts
* Heat maps
* CSV export

---

# 8️⃣ Performance Expectations (translate for students)

From PRD:

| Feature    | Target  |
| ---------- | ------- |
| Scan write | < 1 sec |
| Dashboard  | < 3 sec |
| Concurrent | 1000    |
| Uptime     | 99.5%   |

Make them:

✅ index foreign keys
✅ use pagination
✅ cache dashboard queries

---

# 9️⃣ Tutor “explain it in class” version

Use these:

### Attendance System = Ledger

> "Every scan is a financial transaction, not an event."

---

### QR codes = tokens

> "QR codes are signed tickets, not pictures."

---

### Admin access

> "If admins can edit records, they must also leave fingerprints."

---

### Roles

> "Access control is not UI — it’s backend authority."

---

### Logging

> "No logs = no truth."

---

# 🔟 Suggested Capstone Milestones (for students)

### Week 1

User + Auth

### Week 2

QR generation + scanner

### Week 3

Attendance logging

### Week 4

Admin dashboard

### Week 5

Reports

### Week 6

Security & polishing

---

# ✅ What I can provide next (if you want):

✅ Full ER diagram
✅ .NET starter repo structure
✅ Sample endpoints
✅ Postman collection
✅ Student grading rubric
✅ API DTOs
✅ Minimal Flutter scan code
✅ JWT auth boilerplate
✅ Database migration scripts

---