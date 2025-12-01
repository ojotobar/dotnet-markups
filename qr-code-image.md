### 1. Dependencies

You’ll need these NuGet packages:

```bash
dotnet add package QRCoder
dotnet add package System.IdentityModel.Tokens.Jwt
```

---

### 2. Generate a Signed Token

We’ll use **JWT** with a short expiry. This token can then be encoded in a QR code.

```csharp
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

public static class TokenHelper
{
    private static readonly string SecretKey = "your-very-strong-secret-key"; // store securely

    public static string GenerateSignedToken(string userId, int validMinutes = 5)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(SecretKey));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: "YourApp",
            audience: "YourAppUsers",
            claims: new[] { new Claim("userId", userId) },
            expires: DateTime.UtcNow.AddMinutes(validMinutes),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public static ClaimsPrincipal? ValidateToken(string token)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.UTF8.GetBytes(SecretKey);

        try
        {
            var principal = tokenHandler.ValidateToken(token, new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidIssuer = "YourApp",
                ValidAudience = "YourAppUsers",
                IssuerSigningKey = new SymmetricSecurityKey(key)
            }, out var validatedToken);

            return principal;
        }
        catch
        {
            return null; // token invalid or expired
        }
    }
}
```

---

### 3. Generate QR Code

We’ll embed the JWT inside the QR code using **QRCoder**:

```csharp
using QRCoder;
using System.Drawing;

public static class QRCodeHelper
{
    public static Bitmap GenerateQRCode(string token)
    {
        using var qrGenerator = new QRCodeGenerator();
        using var qrData = qrGenerator.CreateQrCode(token, QRCodeGenerator.ECCLevel.Q);
        using var qrCode = new QRCode(qrData);
        return qrCode.GetGraphic(20); // 20 = pixels per module
    }
}
```

You can then return this QR as a PNG in a controller:

```csharp
[HttpGet("qrcode/{userId}")]
public IActionResult GetQRCode(string userId)
{
    var token = TokenHelper.GenerateSignedToken(userId, 5); // 5 minutes expiry
    using var qr = QRCodeHelper.GenerateQRCode(token);
    using var ms = new MemoryStream();
    qr.Save(ms, System.Drawing.Imaging.ImageFormat.Png);
    return File(ms.ToArray(), "image/png");
}
```

---

### 4. Validating the QR Code

When the QR code is scanned, your backend extracts the JWT from the QR and validates it:

```csharp
[HttpPost("scan")]
public IActionResult ScanQRCode([FromBody] string token)
{
    var principal = TokenHelper.ValidateToken(token);
    if (principal == null)
        return Unauthorized("QR code is invalid or expired.");

    var userId = principal.FindFirst("userId")?.Value;
    return Ok($"QR scanned successfully for user {userId}");
}
```

---

✅ This setup ensures:

* The QR is **time-limited** (expiring QR enforcement).
* The token is **signed** (cannot be tampered).
* Scanning the QR immediately validates it securely.

---
Perfect! Let’s design a **fully self-contained demo** for your project:

* One page can **generate a QR** for a “user” (teacher/student simulation).
* Another page (or the same page on a different device/browser) can **scan the QR** live.
* Everything is **time-limited and signed** for security.

---

## 1️⃣ Backend: ASP.NET Core

We can simplify the backend to **serve both pages** and handle the JWT scan.

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
public class QRDemoController : ControllerBase
{
    [HttpGet("/demo")]
    public IActionResult DemoPage()
    {
        return PhysicalFile("wwwroot/demo.html", "text/html"); // serve static HTML page
    }

    [HttpPost("/api/generate")]
    public IActionResult GenerateQRCode([FromBody] string userId)
    {
        var token = TokenHelper.GenerateSignedToken(userId, 5); // 5 min expiry
        var qrBase64 = QRCodeHelper.GenerateQRCodeBase64(token);
        return Ok(new { qr = qrBase64, token });
    }

    [HttpPost("/api/scan")]
    public IActionResult ScanQRCode([FromBody] string token)
    {
        var principal = TokenHelper.ValidateToken(token);
        if (principal == null)
            return Unauthorized(new { message = "QR code is invalid or expired." });

        var userId = principal.FindFirst("userId")?.Value;
        return Ok(new { message = $"QR scanned successfully for user {userId}" });
    }
}
```

---

## 2️⃣ Frontend: Single HTML for Generate + Scan

Save as `wwwroot/demo.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>QR Demo Project</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        body { font-family: sans-serif; text-align: center; }
        img { max-width: 300px; margin: 10px; }
    </style>
</head>
<body>
    <h1>QR Demo</h1>

    <div>
        <h2>Generate QR</h2>
        <input type="text" id="userId" placeholder="Enter User ID" />
        <button onclick="generateQR()">Generate</button>
        <div id="qrContainer"></div>
        <p id="expiryNotice"></p>
    </div>

    <hr>

    <div>
        <h2>Scan QR</h2>
        <div id="reader" style="width:300px;"></div>
        <p id="scanResult"></p>
    </div>

    <script>
        async function generateQR() {
            const userId = document.getElementById('userId').value;
            if (!userId) return alert('Enter a user ID');

            const res = await fetch('/api/generate', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(userId)
            });
            const data = await res.json();
            document.getElementById('qrContainer').innerHTML = `<img src="${data.qr}" />`;
            document.getElementById('expiryNotice').innerText = 'Expires in 5 minutes';
        }

        function onScanSuccess(decodedText) {
            fetch('/api/scan', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(decodedText)
            })
            .then(res => res.json())
            .then(data => {
                document.getElementById('scanResult').innerText = data.message;
            })
            .catch(err => {
                document.getElementById('scanResult').innerText = 'Scan failed';
                console.error(err);
            });
        }

        var html5QrcodeScanner = new Html5Qrcode("reader");
        html5QrcodeScanner.start(
            { facingMode: "environment" },
            { fps: 10, qrbox: 250 },
            onScanSuccess
        );
    </script>
</body>
</html>
```

---

### ✅ Features

1. **Generate QR** for a specific user ID.
2. **Live scan** using camera (mobile or desktop).
3. **Signed token** ensures security.
4. **Expiring QR** after 5 minutes.
5. Fully **self-contained demo** for classroom use.

---
