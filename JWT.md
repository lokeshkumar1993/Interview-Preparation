# Detailed JWT Authentication Interview Question

---

# Question — JWT Authentication in ASP.NET Core

---

## What is JWT?

JWT stands for **JSON Web Token**.

It is a compact, secure token format used for:
- Authentication
- Authorization
- Secure API communication

JWT is commonly used in:
- ASP.NET Core APIs
- Microservices
- SPA applications (Angular/React)
- Mobile applications

---

# Why JWT is Used?

Traditional session-based authentication stores session on server.

JWT is:
- Stateless
- Scalable
- Lightweight
- Suitable for distributed systems

---

# JWT Structure

JWT consists of 3 parts separated by dots (`.`)

```text
xxxxx.yyyyy.zzzzz
```

Structure:

```text
Header.Payload.Signature
```

---

# Example JWT Token

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiIxMjMiLCJuYW1lIjoiSm9obiJ9
.
abcxyzsignature
```

---

# JWT Components

---

## 1. Header

Contains:
- Algorithm
- Token type

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## 2. Payload

Contains claims (user information).

Example:

```json
{
  "sub": "101",
  "name": "John",
  "role": "Admin"
}
```

---

## Common Claims

| Claim | Meaning |
|---|---|
| sub | Subject/UserId |
| name | Username |
| role | User role |
| exp | Expiration time |
| iss | Issuer |
| aud | Audience |

---

## 3. Signature

Used to verify token integrity.

Generated using:
- Header
- Payload
- Secret key

---

# JWT Authentication Flow

---

## Step 1 — User Logs In

User sends:
- username
- password

to API.

---

## Step 2 — Server Validates User

API checks credentials from:
- Database
- Identity provider

---

## Step 3 — JWT Token Generated

Server generates JWT containing:
- UserId
- Roles
- Claims

---

## Step 4 — Token Returned

API returns JWT token to client.

Example:

```json
{
  "token": "eyJhbGciOi..."
}
```

---

## Step 5 — Client Stores Token

Usually stored in:
- LocalStorage
- SessionStorage
- HttpOnly Cookie

---

## Step 6 — Client Sends Token

In every request:

```http
Authorization: Bearer eyJhbGc...
```

---

## Step 7 — API Validates Token

Checks:
- Signature
- Expiration
- Issuer
- Audience

---

# Advantages of JWT

---

## Advantages

- Stateless
- Scalable
- No server session storage
- Good for microservices
- Cross-platform support

---

# Disadvantages of JWT

---

## Disadvantages

- Cannot invalidate easily before expiry
- Larger token size
- Sensitive data should not be stored

---

# JWT vs Session Authentication

| JWT | Session |
|---|---|
| Stateless | Stateful |
| Stored client-side | Stored server-side |
| Better scalability | Better control |
| Suitable for APIs | Suitable for traditional web apps |

---

# What is Refresh Token?

Access token usually has short expiry.

Refresh token is used to:
- Generate new access token
- Avoid forcing user login repeatedly

---

# Access Token vs Refresh Token

| Access Token | Refresh Token |
|---|---|
| Short-lived | Long-lived |
| Sent with API calls | Used to get new token |
| Contains claims | Stored securely |

---

# Where Should JWT Be Stored?

---

## Best Practice

Use:
- HttpOnly secure cookies

Avoid:
- LocalStorage (XSS risk)

---

# What Happens if JWT Expires?

API returns:

```http
401 Unauthorized
```

Client should:
- Use refresh token
- Get new access token

---

# ASP.NET Core JWT Implementation

---

# Step 1 — Install Package

```powershell
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
```

---

# Step 2 — Add JWT Settings in appsettings.json

```json
{
  "Jwt": {
    "Key": "ThisIsVerySecretKey12345",
    "Issuer": "MyApp",
    "Audience": "MyUsers",
    "DurationInMinutes": 60
  }
}
```

---

# Step 3 — Configure JWT Authentication

## Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(
    JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
    options.TokenValidationParameters =
        new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,

            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],

            IssuerSigningKey =
                new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes(
                        builder.Configuration["Jwt:Key"]))
        };
});

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

# Important Interview Question

## Why `UseAuthentication()` before `UseAuthorization()`?

Because:
1. User identity must be established first
2. Then authorization checks permissions

Wrong order causes authorization failure.

---

# Step 4 — Generate JWT Token

## TokenService.cs

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public class TokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(string username)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, username),
            new Claim(ClaimTypes.Role, "Admin")
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(
                _configuration["Jwt:Key"]));

        var credentials =
            new SigningCredentials(
                key,
                SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.Now.AddMinutes(60),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler()
            .WriteToken(token);
    }
}
```

---

# Step 5 — Login API

## AuthController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly TokenService _tokenService;

    public AuthController(TokenService tokenService)
    {
        _tokenService = tokenService;
    }

    [HttpPost("login")]
    public IActionResult Login(LoginModel model)
    {
        if(model.Username == "admin"
           && model.Password == "123")
        {
            var token =
                _tokenService.GenerateToken(model.Username);

            return Ok(new
            {
                token = token
            });
        }

        return Unauthorized();
    }
}
```

---

# Step 6 — Protect API using [Authorize]

```csharp
[Authorize]
[HttpGet]
public IActionResult GetUsers()
{
    return Ok("Protected API");
}
```

---

# Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
```

Only Admin users can access.

---

# Access User Claims

```csharp
var username = User.Identity.Name;

var role = User.FindFirst(ClaimTypes.Role)?.Value;
```

---

# Common JWT Interview Questions

---

# Question — Why JWT is Stateless?

Because server does not store session.

All user information exists inside token.

---

# Question — Can JWT be tampered?

Payload can be decoded.

But if modified:
- Signature validation fails

---

# Question — Is JWT encrypted?

No.

JWT is encoded, not encrypted.

Sensitive data should not be stored.

---

# Question — Difference between Authentication and Authorization

| Authentication | Authorization |
|---|---|
| Who are you? | What can you access? |

---

# Question — Why use HTTPS with JWT?

Without HTTPS:
- Token can be intercepted
- Security risk

---

# Question — What happens if secret key changes?

Previously issued tokens become invalid.

---

# Question — Which algorithm is commonly used?

```text
HS256
```

---

# Question — Can JWT be revoked?

Not easily.

Common approaches:
- Blacklisting
- Short expiry
- Refresh tokens

---

# Microservices JWT Flow

In microservices:
- Auth service generates token
- Other services validate token
- No centralized session needed

This improves scalability.

---

# Common Mistakes Candidates Make

---

## Weak Candidates Usually:

- Cannot explain JWT structure
- Confuse OAuth and JWT
- Don't know middleware order
- Cannot explain refresh token
- Don't know token validation flow

---

# Strong Candidate Indicators

---

## Strong Candidates Explain:

- Authentication flow clearly
- Claims-based authorization
- Token validation
- Refresh token mechanism
- Stateless architecture
- Security concerns

---
