Perfect. Let’s do this **properly, plainly, and defensibly**—the way backend code should look when auditors, future-you, and annoyed tutors come knocking.

I’ll give you:

1. **JWT structure (what goes in the QR)**
2. **QR regeneration endpoint (C# / Minimal API style)**
3. **JWT generation + validation notes**

No fluff. No magic.

---

## 1️⃣ JWT structure (QR payload)

This JWT is **NOT authentication**.
It’s a **short-lived session access token**.

### Claims (keep it boring and explicit)

```json
{
  "iss": "AttendanceSystem",
  "aud": "QrAttendance",
  "sub": "session-qr",
  "sid": "SESSION_ID",
  "tid": "TOKEN_ID",
  "iat": 1736077200,
  "exp": 1736077260
}
```

### What each claim means (important)

| Claim     | Purpose                                       |
| --------- | --------------------------------------------- |
| `sid`     | Session ID (attendance session)               |
| `tid`     | Unique token ID (prevents replay diagnostics) |
| `iat`     | Issued-at (server time)                       |
| `exp`     | Expiry (≈ 60s later)                          |
| `iss/aud` | Extra validation safety                       |

⚠️ **Do NOT include user info.**
QR proves *session presence*, not identity.

---

## 2️⃣ Endpoint: regenerate QR every 60 seconds

### Route

```http
GET /sessions/{sessionId}/qr
```

### Requirements

* Caller must be **authenticated**
* Must be **session owner (tutor/admin)**
* Session must be **Active**
* Current time must be **before endTime**

---

### Minimal API – C# (.NET)

```csharp
app.MapGet("/sessions/{sessionId:guid}/qr",
async (
    Guid sessionId,
    HttpContext http,
    ISessionRepository sessions,
    IJwtTokenService jwtService
) =>
{
    var userId = http.User.GetUserId(); // your auth extension

    var session = await sessions.GetByIdAsync(sessionId);
    if (session == null)
        return Results.NotFound();

    if (!session.IsActive)
        return Results.StatusCode(StatusCodes.Status410Gone);

    if (session.StartedByUserId != userId && !http.User.IsInRole("Admin"))
        return Results.Forbid();

    var now = DateTime.UtcNow;
    if (now > session.EndTimeUtc)
        return Results.StatusCode(StatusCodes.Status410Gone);

    var token = jwtService.GenerateQrToken(session.Id, ttlSeconds: 60);

    return Results.Ok(new
    {
        qrToken = token,
        expiresInSeconds = 60
    });
})
.RequireAuthorization();
```

Old-school rule applied:
**No session mutation. Ever.**

---

## 3️⃣ JWT generation service

### Interface

```csharp
public interface IJwtTokenService
{
    string GenerateQrToken(Guid sessionId, int ttlSeconds);
}
```

---

### Implementation (HMAC SHA256)

```csharp
public sealed class JwtTokenService : IJwtTokenService
{
    private readonly JwtSecurityTokenHandler _handler = new();
    private readonly byte[] _key;

    public JwtTokenService(IConfiguration config)
    {
        _key = Encoding.UTF8.GetBytes(config["Jwt:QrSecret"]);
    }

    public string GenerateQrToken(Guid sessionId, int ttlSeconds)
    {
        var now = DateTime.UtcNow;

        var claims = new List<Claim>
        {
            new("sid", sessionId.ToString()),
            new("tid", Guid.NewGuid().ToString()),
            new(JwtRegisteredClaimNames.Sub, "session-qr"),
            new(JwtRegisteredClaimNames.Iat,
                EpochTime.GetIntDate(now).ToString(),
                ClaimValueTypes.Integer64)
        };

        var token = new JwtSecurityToken(
            issuer: "AttendanceSystem",
            audience: "QrAttendance",
            claims: claims,
            notBefore: now,
            expires: now.AddSeconds(ttlSeconds),
            signingCredentials: new SigningCredentials(
                new SymmetricSecurityKey(_key),
                SecurityAlgorithms.HmacSha256
            )
        );

        return _handler.WriteToken(token);
    }
}
```

---

## 4️⃣ Validation (used by scan endpoint)

When someone scans:

```csharp
var principal = tokenHandler.ValidateToken(
    qrToken,
    new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidIssuer = "AttendanceSystem",

        ValidateAudience = true,
        ValidAudience = "QrAttendance",

        ValidateLifetime = true,
        ClockSkew = TimeSpan.FromSeconds(10),

        IssuerSigningKey = new SymmetricSecurityKey(_key),
        ValidateIssuerSigningKey = true
    },
    out _
);

var sessionId = principal.FindFirst("sid")!.Value;
```

Then:

* confirm session exists
* confirm session is active
* confirm user hasn’t already scanned
* mark attendance

---

## 5️⃣ Why this design holds up

* ✅ Screenshots expire
* ✅ No DB writes for QR rotation
* ✅ Stateless & scalable
* ✅ Session timeline stays immutable
* ✅ Easy to audit disputes

This is the same logic factories, exams, and secure facilities use—just cleaner.

---

## Final rule to remember

> **Sessions are historical facts.
> QR tokens are disposable keys.**

If you want next, I can:

* Write the **scan endpoint**
* Add **Redis replay protection**
* Add **rate limiting**
* Design the **attendance DB schema**

Say the word.




Great — thanks for sharing the SAS MVP document. I’ll give you three things:

1️⃣ A clean, practical **Audit Log data model** (aligned with Admin Adjustments & security expectations)
2️⃣ **Implementation documentation for the Admin Adjustments API**
3️⃣ **Implementation documentation for the Analytics & Dashboard API**

I’ll design these so they are:

* auditable
* scalable for reporting
* consistent with QR sessions + attendance model in the file 

---

# ✅ AUDIT LOG MODEL (Recommended)

Admin actions must be traceable, especially for:

* Manual attendance edits
* Admin overrides
* Deletions
* Security-sensitive actions

### 🎯 Core Principles

An audit log should:

✔ capture who performed the action
✔ capture what changed (before + after)
✔ link to the affected record
✔ store timestamps
✔ never be editable or deletable

---

## 📌 AuditLogs Table (Proposed Schema)

```
AuditLogs
---------
Id (GUID)
TimestampUtc (datetime)
ActorUserId (GUID)
ActorRole (string)
ActionType (enum)
EntityType (string)
EntityId (GUID)
ChangeSummary (string)
BeforeData (json)
AfterData (json)
Reason (string)
IpAddress (string)
DeviceInfo (string)
CorrelationId (string)
```

---

## 🧩 ActionType Enum (examples)

```
CREATE
UPDATE
DELETE
ADMIN_OVERRIDE
MANUAL_ATTENDANCE_ADD
MANUAL_ATTENDANCE_EDIT
MANUAL_ATTENDANCE_DELETE
USER_ACCOUNT_UPDATE
ROLE_CHANGE
LOGIN_SUCCESS
LOGIN_FAILED
```

---

## 🧾 Example Audit Record (Admin Attendance Edit)

```
ActionType: MANUAL_ATTENDANCE_EDIT
EntityType: AttendanceLog
EntityId: a1b2-c3d4
ActorUserId: admin-123
Reason: "Correcting missed scan"
BeforeData:
 {
  "status": "Absent",
  "timestamp": "2025-01-15T09:02:00Z"
 }
AfterData:
 {
  "status": "Present",
  "timestamp": "2025-01-15T09:00:30Z"
 }
```

This supports:

* traceability
* rollback analysis
* audit compliance
* dispute resolution

---

# 🟦 ADMIN ADJUSTMENTS API — IMPLEMENTATION DOCUMENTATION

From the story:

> Add / Edit / Delete attendance
> Require admin reason
> Log to Audit Trail 

This API exists for:

* correcting missed scans
* resolving disputes
* adjusting wrong records

It must never silently modify data.

---

## 🧭 API Security Requirements

* Admin-only roles
* JWT + role claim required
* Every adjustment logs to AuditLogs
* Changes should be **immutable + reversible**

---

## 📌 Endpoint Structure

### 1️⃣ Add Attendance (Manual Create)

```
POST /api/attendance/admin/add
```

Request:

```
{
  "userId": "GUID",
  "timestamp": "2025-01-10T09:00:00Z",
  "status": "Present",
  "reason": "User forgot to scan"
}
```

Response:

```
201 Created
{
  "attendanceId": "GUID",
  "status": "Present"
}
```

Audit Entry Created ✔

---

### 2️⃣ Edit Attendance

```
PUT /api/attendance/admin/{attendanceId}
```

Request:

```
{
  "status": "Late",
  "timestamp": "2025-01-10T09:15:00Z",
  "reason": "Adjusted after review"
}
```

Backend must:

* fetch old record
* store full **BeforeData**
* apply new values
* store full **AfterData**
* write AuditLog

---

### 3️⃣ Delete Attendance

```
DELETE /api/attendance/admin/{attendanceId}
```

Request:

```
{
  "reason": "Duplicate scan"
}
```

Policy Recommendation:

* record should be **soft-deleted**
* status becomes `"Deleted"`
* AuditLog retains full record

---

## 🧾 Adjustment Audit Log Example

```
ActionType: MANUAL_ATTENDANCE_DELETE
ActorUserId: admin-22
EntityType: AttendanceLog
EntityId: 8844-A22
Reason: "Duplicate QR scan"
BeforeData: { full attendance record }
AfterData: { "deleted": true }
```

This matches MVP expectations:

✔ always require reason
✔ log all edits
✔ trace accountability

---

# 🟩 ANALYTICS & DASHBOARD API — IMPLEMENTATION DOCUMENTATION

From the story:

> Daily metrics
> Staff vs Students
> Department breakdown
> Trends 

The API supports:

* dashboard tiles
* charts
* time-series reporting

---

## 📌 Data Sources

Primary tables:

* AttendanceLogs
* Users
* Departments
* QRSession
* (optional) Materialized views for performance

---

## 📊 Metrics Returned

✔ Total attendance today
✔ Present vs absent breakdown
✔ Staff vs student ratio
✔ Departmental attendance
✔ Trends across days

---

## 🧭 Endpoints

### 1️⃣ Dashboard Summary

```
GET /api/analytics/dashboard/summary?date=2025-01-10
```

Response:

```
{
  "date": "2025-01-10",
  "totalScans": 842,
  "present": 790,
  "absent": 52,
  "late": 38,
  "staffPresent": 120,
  "studentsPresent": 670
}
```

---

### 2️⃣ Department Breakdown

```
GET /api/analytics/dashboard/departments?date=2025-01-10
```

Response:

```
[
  { "department": "Computer Science", "present": 240, "absent": 18 },
  { "department": "Business Admin", "present": 180, "absent": 10 }
]
```

---

### 3️⃣ Attendance Trends

```
GET /api/analytics/trends?from=2025-01-01&to=2025-01-10
```

Response:

```
[
  { "date": "2025-01-01", "present": 710 },
  { "date": "2025-01-02", "present": 742 },
  { "date": "2025-01-03", "present": 780 }
]
```

---

## ⚡ Performance Recommendation

For large datasets:

* create **daily rollup summaries**
* or materialized analytics views
* refresh using background jobs

This keeps dashboard queries fast.

---

# 🎯 Next — I can also provide

If useful, I can create:

✔ SQL schema definitions for AuditLogs & AttendanceLogs
✔ C# / .NET or Node backend controller stubs
✔ Stored procedure or EF-Core query design
✔ Event-sourced adjustment history approach
✔ Admin review + approval workflow

---

# 🤝 Tell me how you want to continue

Which would you like next?

👉 C# API implementation stubs
👉 Database migration scripts
👉 ER diagram for audit + attendance
👉 Request/response swagger spec
👉 Role-based permission rules

Just say the word — I’ll build the next layer.
