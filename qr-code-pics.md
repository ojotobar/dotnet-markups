# ✅ What your current API does

This part:

```csharp
return Ok(new
{
    qrValue = token,
    expiresAt = session.ExpiresAt
});
```

Means:

* ✅ You get the encoded value
* ❌ You do NOT get a QR image
* ❌ Browser / phone will NOT automatically show a QR code

---

# ✅ You have TWO OPTIONS

## OPTION 1 — Frontend generates QR (Recommended for teaching)

Return `qrValue` and let the UI generate QR using a library.

### Example in Angular / JS

```html
<qrcode [qrdata]="qrValue" width="256"></qrcode>
```

Or plain JS:

```html
<img src="https://api.qrserver.com/v1/create-qr-code/?data=YOUR_VALUE" />
```

### Teaching point:

> Backend decides the truth, frontend decides the presentation.

This is how real systems work.

---

## OPTION 2 — Backend generates the image (Full Server Mode)

If you want the API itself to return the QR image:

### ✅ Install package

```bash
dotnet add package QRCoder
```

---

## ✅ Update Controller to return IMAGE

### Replace `CreateSession` endpoint:

```csharp
using QRCoder;
using System.Drawing;
using System.Drawing.Imaging;

[HttpPost("create")]
public async Task<IActionResult> CreateSession(string department, int minutes = 10)
{
    var session = new AttendanceSession
    {
        Id = Guid.NewGuid(),
        Department = department,
        CreatedBy = "admin",
        ExpiresAt = DateTime.UtcNow.AddMinutes(minutes)
    };

    _db.AttendanceSessions.Add(session);
    await _db.SaveChangesAsync();

    var token = Convert.ToBase64String(
        Encoding.UTF8.GetBytes($"{session.Id}|{session.ExpiresAt}")
    );

    using var qrGenerator = new QRCodeGenerator();
    using var qrCodeData = qrGenerator.CreateQrCode(token, QRCodeGenerator.ECCLevel.Q);
    using var qrCode = new QRCode(qrCodeData);
    using var bitmap = qrCode.GetGraphic(20);

    using var ms = new MemoryStream();
    bitmap.Save(ms, ImageFormat.Png);

    return File(ms.ToArray(), "image/png");
}
```

---

## ✅ Result:

Now when you open:

```
POST /api/qr/create?department=ComputerScience
```

In browser or Postman:

✅ You SEE the QR image directly
✅ Can scan from phone camera
✅ Works like real attendance systems

---

# ✅ Which should YOU use for students?

| Approach    | Use when               |
| ----------- | ---------------------- |
| Frontend QR | Teaching frontend      |
| Backend QR  | Teaching API & imaging |
| Both        | Advanced class         |

---

# ✅ Bonus Teaching Hack

Add this text under QR:

> "QR is temporary. Server is eternal." 😎