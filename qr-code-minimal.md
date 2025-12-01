* QR generation endpoint
* Attendance scan endpoint
* Database models
* Minimal validation
* Beginner-friendly architecture

No fluff. Just working code and explanations you can teach from.

---

# ✅ PROJECT STRUCTURE (Simple & Teachable)

```
/Controllers
  ├── QrController.cs
  └── AttendanceController.cs

/Models
  ├── AppUser.cs
  ├── AttendanceSession.cs
  └── AttendanceLog.cs

/Data
  └── AppDbContext.cs
```

---

# ✅ MODELS

## AttendanceSession.cs (QR session)

```csharp
public class AttendanceSession
{
    public Guid Id { get; set; }
    public DateTime ExpiresAt { get; set; }
    public bool IsActive { get; set; } = true;
    public string Department { get; set; } = default!;
    public string CreatedBy { get; set; } = default!; // AdminId
}
```

---

## AttendanceLog.cs

```csharp
public class AttendanceLog
{
    public Guid Id { get; set; }
    public string UserId { get; set; } = default!;
    public Guid SessionId { get; set; }
    public DateTime TimeScanned { get; set; } = DateTime.UtcNow;
}
```

---

# ✅ DB CONTEXT

## AppDbContext.cs

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}

    public DbSet<AttendanceSession> AttendanceSessions => Set<AttendanceSession>();
    public DbSet<AttendanceLog> AttendanceLogs => Set<AttendanceLog>();
}
```

---

# ✅ QR CODE CREATION ENDPOINT

## QrController.cs

```csharp
[ApiController]
[Route("api/qr")]
public class QrController : ControllerBase
{
    private readonly AppDbContext _db;

    public QrController(AppDbContext db)
    {
        _db = db;
    }

    [HttpPost("create")]
    public async Task<IActionResult> CreateSession(string department, int minutes = 10)
    {
        var session = new AttendanceSession
        {
            Id = Guid.NewGuid(),
            Department = department,
            CreatedBy = "admin",
            ExpiresAt = DateTime.UtcNow.AddMinutes(minutes),
            IsActive = true
        };

        _db.AttendanceSessions.Add(session);
        await _db.SaveChangesAsync();

        var token = Convert.ToBase64String(
            Encoding.UTF8.GetBytes($"{session.Id}|{session.ExpiresAt}")
        );

        return Ok(new
        {
            qrValue = token,
            expiresAt = session.ExpiresAt
        });
    }
}
```

---

### ✅ What the QR actually contains

It encodes:

```
SessionId | ExpiryTime
```

Teach students:

> "QR is just text — the backend decides truth."

---

# ✅ ATTENDANCE SCAN ENDPOINT

## AttendanceController.cs

```csharp
[ApiController]
[Route("api/attendance")]
public class AttendanceController : ControllerBase
{
    private readonly AppDbContext _db;

    public AttendanceController(AppDbContext db)
    {
        _db = db;
    }

    [HttpPost("scan")]
    public async Task<IActionResult> Scan(string qrValue, string userId)
    {
        var decoded = Encoding.UTF8.GetString(Convert.FromBase64String(qrValue));
        var parts = decoded.Split('|');

        var sessionId = Guid.Parse(parts[0]);
        var expiry = DateTime.Parse(parts[1]);

        if (DateTime.UtcNow > expiry)
            return BadRequest("QR Code expired");

        var session = await _db.AttendanceSessions.FindAsync(sessionId);

        if (session == null || !session.IsActive)
            return BadRequest("Invalid session");

        var alreadyMarked = await _db.AttendanceLogs
            .AnyAsync(a => a.SessionId == sessionId && a.UserId == userId);

        if (alreadyMarked)
            return BadRequest("Already marked attendance");

        var log = new AttendanceLog
        {
            Id = Guid.NewGuid(),
            SessionId = sessionId,
            UserId = userId
        };

        _db.AttendanceLogs.Add(log);
        await _db.SaveChangesAsync();

        return Ok("Attendance recorded");
    }
}
```

---

# ✅ MINIMUM SECURITY RULES (Teach these!)

### Students MUST implement:

✅ No duplicate attendance
✅ No expired QR
✅ No invalid QR
✅ Session must exist
✅ Mandatory timestamp

---

# ✅ HOW TO TEST (Postman / Swagger)

### Create QR:

```
POST /api/qr/create?department=ComputerScience&minutes=10
```

Response:

```
qrValue: "base64stuff"
```

---

### Scan attendance:

```
POST /api/attendance/scan
Body:
{
   "qrValue": "...",
   "userId": "123"
}
```

---

# ✅ OPTIONAL ADVANCED (Week 6–8 upgrades)

Add later if students are ready:

* JWT authentication
* Geo-location capture
* Device fingerprinting
* Admin override
* Attendance reports
* Daily auto-close of session
* Admin dashboard

---

# ✅ Tutor One-Liners (Use in Class)

### QR:

> "QR is just ink — validation is authority."

### Attendance:

> "Once written, never edited — only audited."

### Backend rule:

> "Frontend may lie. Backend must not."

---

# ✅ 8-WEEK ROADMAP (Beginner Friendly)

| Week | Focus          |
| ---- | -------------- |
| 1    | Auth + Users   |
| 2    | QR creation    |
| 3    | Scanning       |
| 4    | Attendance DB  |
| 5    | Admin controls |
| 6    | Reports        |
| 7    | Security       |
| 8    | Polishing      |

---