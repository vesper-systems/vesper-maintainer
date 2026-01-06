# REQUEST-003: Session Cookie Authentication System

## Goal
Establish session-based authentication flow from vesper-loader → Railway backend → vesper-client with hardcoded credentials for MVP.

## Scope
- vesper-protocol: Define auth contracts (`/auth/login`, `/auth/validate`, session cookie format)
- vesper-backend: Implement auth endpoints with hardcoded credentials, session management, Railway deployment
- vesper-loader: Login UI → POST to backend → store session cookie → launch client
- vesper-client: Accept session cookie from loader, validate on startup

## Constraints
- Must use session cookies (not JWT or API keys)
- Credentials hardcoded in backend for MVP
- Session cookies must be HttpOnly and Secure (production)
- No persistent user database for MVP
- All repos must conform to vesper-protocol auth specifications

## Components

### vesper-protocol AI Instructions
```
CREATE: docs/auth/SESSION_AUTH.md

Define:
1. POST /auth/login endpoint contract
   - Request: { username: string, password: string }
   - Response: Set-Cookie header with session token
   - Response body: { success: bool, message: string, expiresAt: timestamp }

2. POST /auth/validate endpoint contract
   - Request: Cookie header with session token
   - Response: { valid: bool, username: string, expiresAt: timestamp }

3. Session cookie format
   - Name: "vesper_session"
   - HttpOnly: true
   - Secure: true (production), false (development)
   - SameSite: Strict
   - Max-Age: 86400 (24 hours)
   - Path: /
   - Value: base64(sessionId:timestamp:hmacSignature)

4. Error codes
   - 401: Invalid credentials or missing cookie
   - 403: Session expired or invalid signature
   - 429: Rate limit exceeded (max 5 login attempts per minute)
   - 500: Server error

5. Security requirements
   - HMAC-SHA256 signature on session tokens
   - Session expiration enforced server-side
   - Automatic session cleanup (remove expired every hour)
```

### vesper-backend AI Instructions
```
IMPLEMENT: Session-based authentication system for Railway deployment

Requirements:

1. Environment Configuration
   - AUTH_USERNAME: Hardcoded username (from env var)
   - AUTH_PASSWORD: Hardcoded password (from env var)
   - SESSION_SECRET: HMAC signing key (from env var, generate random 32-byte hex)
   - NODE_ENV: production/development
   - PORT: 3000 (default)

2. POST /auth/login
   - Validate credentials against env vars
   - Rate limit: 5 attempts per IP per minute
   - On success:
     * Generate session ID (UUID v4)
     * Create HMAC signature: HMAC-SHA256(sessionId + timestamp, SESSION_SECRET)
     * Store session in memory: Map<sessionId, {username, createdAt, expiresAt}>
     * Set HttpOnly cookie with base64(sessionId:timestamp:signature)
     * Return: { success: true, message: "Login successful", expiresAt: timestamp }
   - On failure:
     * Return 401: { success: false, message: "Invalid credentials" }

3. POST /auth/validate
   - Parse vesper_session cookie
   - Decode base64, split into sessionId:timestamp:signature
   - Verify HMAC signature
   - Check session exists and not expired
   - Return: { valid: true, username: string, expiresAt: timestamp }
   - On invalid: { valid: false, message: "Session expired or invalid" }

4. Session Management
   - In-memory store: Map<sessionId, SessionData>
   - Background cleanup task (every 1 hour, remove expired sessions)
   - Session expiration: 24 hours from creation

5. Railway Deployment
   - Create Procfile: web: node src/index.js
   - Create railway.json with build config
   - Environment variables configured in Railway dashboard
   - Health check endpoint: GET /health returns { status: "ok", uptime: seconds }

6. Dependencies
   - express (web framework)
   - cookie-parser (cookie parsing)
   - crypto (HMAC signing, built-in)
   - uuid (session ID generation)
   - express-rate-limit (rate limiting)

7. Error Handling
   - All endpoints wrapped in try-catch
   - Structured error responses
   - Error logging (console.error with timestamp)

8. CORS Configuration
   - Allow localhost for development
   - Restrict to vesper-loader origin in production
```

### vesper-loader AI Instructions
```
IMPLEMENT: Login screen with session cookie storage and client launch

Requirements:

1. JavaFX Login Form UI
   Components:
   - Title label: "Vesper Login"
   - Username TextField
   - Password PasswordField (masked input)
   - Login Button (primary action)
   - Error Label (red text, hidden by default)
   - Loading spinner (hidden by default, shows during request)
   - "Remember Me" checkbox (optional, saves username only)

   Styling:
   - Dark theme (match client UI)
   - Rounded corners (6px)
   - Cobalt accent color on focus
   - Smooth transitions on button hover

2. Login Flow
   On "Login" button click:
   a. Validate inputs (non-empty username/password)
   b. Show loading spinner, disable button
   c. POST to backend /auth/login:
      - URL from config: AUTH_BACKEND_URL (default: http://localhost:3000)
      - Headers: Content-Type: application/json
      - Body: { username: <input>, password: <input> }
   d. Handle response:
      - Success (200): Extract Set-Cookie header → save to file → launch client
      - Failure (401/403): Show error message, re-enable form
      - Network error: Show "Cannot connect to server" error
   e. Hide spinner, re-enable button

3. Session Cookie Storage
   - Save cookie to: %TEMP%\vesper_session.txt (Windows)
   - File format (single line): vesper_session=<token_value>
   - File permissions: Read-only for current user
   - Delete file on application exit (shutdown hook)

4. Client Launch
   - Locate vesper-client.jar (same directory as loader JAR)
   - Launch with JVM args:
     * -Dvesper.sessionFile=%TEMP%\vesper_session.txt
     * -Dvesper.backendUrl=<AUTH_BACKEND_URL>
   - Inherit system properties and classpath
   - Detached process (loader exits after launch)
   - Wait 2 seconds, verify client process started successfully

5. Configuration
   - config.properties file:
     * auth.backend.url=http://localhost:3000 (default)
     * client.jar.path=./vesper-client.jar
     * remember.username=false (default)
   - Override via command-line args: --backend-url=<url>

6. Error Handling
   - Network errors: "Cannot connect to authentication server"
   - Invalid credentials: "Username or password incorrect"
   - Client JAR not found: "Client application not found. Please reinstall."
   - Session save failure: "Could not save session. Check disk permissions."

7. Dependencies
   - JavaFX 17+ (UI framework)
   - Java 17+ (HttpClient for HTTP requests)
   - Gson or Jackson (JSON parsing)

8. Security
   - Never log passwords
   - Clear password field after failed attempt
   - Session file deleted on exit (no persistence)
```

### vesper-client AI Instructions
```
IMPLEMENT: Session validation on startup and authenticated state management

Requirements:

1. Startup Session Validation
   On client initialization:
   a. Read session file path from system property: vesper.sessionFile
   b. If file exists:
      - Read cookie value from file
      - POST to backend /auth/validate with cookie in header
      - If valid: Store username, set authenticated state
      - If invalid: Show error screen, exit gracefully
   c. If file missing:
      - Show error: "Authentication required. Please restart via launcher."
      - Exit after 5 seconds

2. Authenticated State Management
   - Store in singleton: AuthManager
   - Properties:
     * username: String
     * sessionCookie: String
     * expiresAt: long (timestamp)
     * isAuthenticated: boolean
   - Methods:
     * init(sessionFilePath: String): boolean
     * getUsername(): String
     * isValid(): boolean (checks expiration)
     * invalidate(): void (clears session, exits client)

3. UI Integration
   - Display username in top-right corner of main menu
   - Show session expiration countdown (optional)
   - Logout button: Clears session file, exits client

4. Session Refresh (Future)
   - Background task checks session validity every 5 minutes
   - If session expired, show notification and exit
   - Optional: Auto-refresh session before expiration

5. Error Handling
   - Network errors during validation: Retry 3 times, then exit
   - Invalid session: Clear file, show error, exit
   - Backend unreachable: Allow offline mode (if implemented)

6. Dependencies
   - HTTP client (OkHttp or Java 11+ HttpClient)
   - JSON parser (Gson or Jackson)

7. Security
   - Never display full session token in UI
   - Session file deleted on client exit
   - No caching of credentials (only session token)
```

## Definition of Done
- vesper-protocol defines SESSION_AUTH.md contract
- vesper-backend implements `/auth/login` and `/auth/validate` endpoints
- vesper-backend deployed to Railway with environment variables configured
- vesper-loader implements login UI and client launch flow
- vesper-client validates session on startup
- Integration test: Login via loader → validate in client → access protected features
- Session expiration handled gracefully (logout after 24 hours)
- Error scenarios tested (invalid credentials, expired session, network failure)

## Affected Repos
- vesper-protocol
- vesper-backend
- vesper-loader
- vesper-client

## Parallelizable Tasks
- After protocol definition complete: Backend, Loader, Client can implement in parallel

## Sequential Constraints
1. vesper-protocol must define SESSION_AUTH.md first
2. Backend, Loader, Client implement in parallel
3. Integration testing after all components complete

## Notes
- Credentials: AUTH_USERNAME=jazzer, AUTH_PASSWORD=meow (set in Railway env)
- Port: 3000 for local development; on Railway, bind to the injected PORT env var and listen on 0.0.0.0
- If Railway requires manual port configuration, set service port to 3000 and ensure the app listens on 0.0.0.0:3000
- Session storage is in-memory (no database for MVP)
- Railway backend URL to be provided after deployment
- Future enhancement: User registration, password hashing, persistent storage

## Access Controls (Sensitive Info)
- Backend URL visibility is limited to vesper-loader and vesper-client at runtime only.
- vesper-protocol and vesper-backend AIs must NOT publish, log, or commit the actual Railway URL.
- No repository may contain the real Railway URL in source, docs, or examples; use placeholder `<VESPER_BACKEND_URL>`.
- Logs must redact the host: print "[redacted-host]" when referencing endpoints.

## Backend URL Distribution
- Maintainer provides the actual Railway URL out-of-band; it is injected into loader via config and passed to client via `-Dvesper.backendUrl`.
- Loader config file (not committed): `auth.backend.url=<VESPER_BACKEND_URL>`.
- Client receives the URL only via the loader JVM property; client must not persist or log the URL.
- If environment variables are used, name `VESPER_BACKEND_URL` and load at runtime; do not commit defaults.

---
Approval:
   Status: APPROVED
  AutoCommit: false
  CommitMessage: "REQUEST-003: Session cookie authentication system"
  Scope:
    - vesper-protocol
    - vesper-backend
    - vesper-loader
    - vesper-client
