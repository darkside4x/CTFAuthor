# CTF Author Platform 🛡️

A **Zero-Trust** professional directory for Capture The Flag (CTF) challenge creators. Built with security as the absolute top priority.

## 🎯 Project Overview

CTF Author is a reputation-based platform where CTF organizers can discover and contact verified challenge authors. The platform implements multiple security layers to protect user data and prevent abuse.

## 🔐 Core Security Features

### 1. **UUID-Only Architecture**
- All database models use UUID primary keys
- No integer IDs exposed in URLs or APIs
- Prevents enumeration attacks

### 2. **Silent Verification Protocol**
- Admins can verify authors
- System records WHO verified (for audit trails)
- Authors NEVER see who verified them
- `verified_by` field is NEVER exposed in API responses

### 3. **Anti-Scraper Contact System**
- Contact info (email, Discord) is NOT in HTML source
- Fetched via rate-limited AJAX endpoint (5 requests/minute per IP)
- Returns 403 Forbidden if rate limit exceeded
- Prevents automated scraping of contact information

### 4. **Admin Honeypot**
- Real admin panel hidden at secret URL: `secure-command-center-uuid/`
- Standard `/admin/` URL is a TRAP using `django-admin-honeypot`
- Logs all attempted intrusions
- Admins must use 2FA (TOTP)

### 5. **Input Sanitization**
- All user input sanitized with `bleach` before saving
- Prevents XSS attacks in display names and bios
- HTML tags stripped automatically

## 📁 Project Structure

```
CTFAuthor/
├── users/                      # Main application
│   ├── models.py              # User, AuthorProfile, PricingMatrix
│   ├── views.py               # Views with rate limiting
│   ├── admin.py               # Admin panel (verified_by hidden from non-superusers)
│   ├── serializers.py         # Safe serializers (no sensitive fields)
│   ├── urls.py                # URL routing
│   └── templates/
│       └── users/
│           └── author_profile.html  # Secure template with AJAX contact fetch
├── ctfauthor/                 # Project settings
│   ├── settings.py           # Django settings
│   └── urls.py               # Honeypot + real admin configuration
├── requirements.txt          # Python dependencies
├── .env.example             # Environment variables template
└── .github/
    └── copilot-instructions.md  # AI agent guidelines
```

## 🚀 Quick Start

### 1. **Clone and Setup Environment**

```bash
# Clone the repository
git clone <repository-url>
cd CTFAuthor

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment variables
cp .env.example .env
# Edit .env with your configuration
```

### 2. **Database Setup**

```bash
# Run migrations
python manage.py migrate

# Create superuser (Admin)
python manage.py createsuperuser
# IMPORTANT: Use email-based authentication
```

### 3. **Run Development Server**

```bash
python manage.py runserver
```

### 4. **Access Admin Panel**

⚠️ **SECURITY WARNING**: Do NOT use the standard `/admin/` URL in production!

- **Honeypot (Trap):** `http://localhost:8000/admin/` - Logs intrusion attempts
- **Real Admin:** `http://localhost:8000/secure-command-center-a7f3d2e1/` - Use this URL

## 🔒 Security Checklist for Production

- [ ] Change `SECURE_ADMIN_PATH` in `.env` to a random, unguessable string
- [ ] Set `DEBUG=False` in `.env`
- [ ] Configure PostgreSQL database (don't use SQLite)
- [ ] Enable HTTPS/SSL (set `SECURE_SSL_REDIRECT=True`)
- [ ] Configure email backend for Django Allauth
- [ ] Enforce 2FA for all admin accounts
- [ ] Set up fail2ban rules for honeypot admin
- [ ] Review `python manage.py check --deploy` output
- [ ] Configure Sentry or error tracking
- [ ] Whitelist admin IP addresses (if possible)

## 🧪 Testing Security Features

### Test Rate Limiting (Contact Endpoint)

```bash
# Try to request contact info more than 5 times in 1 minute
for i in {1..7}; do
  curl -X POST http://localhost:8000/api/contact/<author-uuid>/
  echo "Request $i"
done
# Requests 6 and 7 should return 403 Forbidden
```

### Test Input Sanitization

1. Create an author profile with HTML in display name: `<script>alert('XSS')</script>Test`
2. Verify that tags are stripped and only `Test` is saved

### Test Silent Verification

1. Admin verifies an author
2. Check database: `verified_by` field is populated
3. Visit author profile page: `verified_by` is NOT visible
4. Check API response: `verified_by` is NOT included

## 🎭 User Roles

| Role | Permissions |
|------|-------------|
| **Visitor** | Search/filter authors, view profiles (no contact info without rate limit check) |
| **Author** | Create profile, set pricing matrix, request verification |
| **Admin** | Access secret admin panel, verify authors, view audit logs |

## 📊 Database Models

### CustomUser
- UUID primary key
- Email-based authentication
- Role: `Author` or `Admin`

### AuthorProfile
- Linked to CustomUser
- Public fields: Display name, bio, country, CTFTime rank
- Sensitive fields: Email (from user), Discord ID
- Audit field: `verified_by` (internal only)

### PricingMatrix
- Linked to AuthorProfile
- Categories: Web, Pwn, Crypto, Reverse, Forensics, OSINT, Misc
- Difficulty: Easy, Medium, Hard
- Price in USD

## 🛠️ Development Commands

```bash
# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run development server
python manage.py runserver

# Run tests
python manage.py test

# Check for security issues
python manage.py check --deploy

# Create new migrations
python manage.py makemigrations
```

## 🔍 Monitoring & Logging

- Django admin honeypot logs all `/admin/` access attempts
- Track these logs for potential security threats
- Consider integrating with fail2ban or similar tools
- Monitor rate limit violations in application logs

## 📝 License

[Your License Here]

## 🤝 Contributing

Security is paramount. All contributions must:
1. Follow the security guidelines in `.github/copilot-instructions.md`
2. Never expose sensitive fields in API responses
3. Always use rate limiting for public endpoints
4. Sanitize all user input

## ⚠️ Security Disclosure

If you discover a security vulnerability, please email [security@example.com] directly. Do NOT open a public issue.

---

**Remember:** This is a ZERO-TRUST platform. Security > Convenience.
