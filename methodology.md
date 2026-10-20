# Assessment Methodology and Practical Procedures

**Course:** Honors Cyber Forensics & Laws (CFL 216H02C501)  
**Project:** Web Application Authentication and Access Control Security Assessment  
**Authors:** Krishna Rathod (16010125801), Riya Vikam (16010125802), Khushi Satam (16010125829)  

---

## 1. General Setup & Environment Configuration

All security assessment activities were performed within an isolated, authorized lab environment using **Burp Suite Community Edition** and the **PortSwigger Web Security Academy**.

### 1.1 Architecture & Interception Setup
- **Interception Proxy:** Burp Suite Community Edition configured to listen on loopback interface `127.0.0.1:8080`.
- **Target Browser:** Chromium-based browser configured to route HTTP/HTTPS traffic through the local proxy.
- **TLS/HTTPS Decryption:** Installed and trusted the PortSwigger CA Certificate (`cacert.der`) in the browser certificate store to inspect encrypted HTTPS traffic without certificate warnings.
- **Scope Restriction:** Strict target scope was defined to match PortSwigger lab instances (`*.web-security-academy.net`) to prevent interception of unintended background network traffic.

---

## 2. Practical 1: Username Enumeration via Different Responses

### 2.1 Objective
Evaluate the login endpoint to determine whether differences in server error responses can be leveraged to enumerate valid user accounts, and subsequently perform targeted credential cracking.

### 2.2 Tool and Lab Used
- **Primary Tool:** Burp Suite Community Edition (Proxy, HTTP History, Intruder)
- **Lab Link:** [PortSwigger Lab: Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

### 2.3 Steps Performed
1. **Baseline Request Capture:**
   - Navigated to the `/login` endpoint.
   - Submitted a baseline login attempt using arbitrary dummy credentials (`username=testuser123`, `password=dummyPassword`).
   - Intercepted the `POST /login` request and observed the default error response: `Invalid username`.

2. **Automated Username Enumeration via Burp Intruder:**
   - Sent the intercepted `POST /login` request to **Burp Intruder**.
   - Set attack type to **Sniper**.
   - Positioned payload markers exclusively around the `username` parameter value:
     ```http
     username=§testuser§&password=dummyPassword
     ```
   - Loaded the candidate username dictionary provided by the lab.
   - Executed the attack and sorted the results by response length, status code, and response body text.
   - Identified an anomaly: one specific username returned a distinct error response (`Incorrect password`), confirming that the account exists on the target system.

3. **Targeted Password Brute-Force:**
   - Updated the Intruder configuration with the identified valid username fixed in the request body.
   - Configured the payload marker on the `password` parameter:
     ```http
     username=confirmed_user&password=§password§
     ```
   - Loaded the candidate passwords dictionary and executed the attack.
   - Observed a `302 Found` redirection response upon sending the valid password, indicating successful authentication.

4. **Validation:**
   - Followed the redirect and established an authenticated session.
   - Verified that the PortSwigger Academy interface registered the lab as solved.

### 2.4 Actual Observed Result
The application returned divergent error strings:
- For non-existent accounts: `"Invalid username"`
- For the valid account: `"Incorrect password"`

This subtle difference allowed positive identification of a valid user account. Using the confirmed username, automated password spraying succeeded with a `302 Found` response. The lab was successfully solved.

### 2.5 Security Impact
- **Account Harvesting:** Allows unauthenticated attackers to build lists of valid usernames and employee identifiers.
- **Targeted Attacks:** Eliminates guessing uncertainty, significantly increasing the success rate and efficiency of brute-force, password-spraying, and credential-stuffing campaigns.

### 2.6 Mitigation
- **Uniform Error Messages:** Return identical, generic error messages for all failed login attempts (e.g., *"Invalid username or password"*), regardless of whether the username exists.
- **Timing Normalization:** Ensure constant-time password hash comparisons to prevent timing side-channel enumeration.
- **Rate Limiting & Account Lockout:** Enforce IP-based rate limiting, progressive delays, or CAPTCHA verification after consecutive failed login attempts.

---

## 3. Practical 2: Insecure Direct Object References (IDOR)

### 3.1 Objective
Determine whether access controls on user profile data rely solely on client-supplied request parameters without verifying server-side session authorization.

### 3.2 Tool and Lab Used
- **Primary Tool:** Burp Suite Community Edition (Proxy, HTTP History, Repeater)
- **Lab Link:** [PortSwigger Lab: User ID controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)

### 3.3 Steps Performed
1. **Authenticated Baseline Mapping:**
   - Logged into the application using valid credentials assigned to the tester (`wiener:peter`).
   - Navigated to the user dashboard at `/my-account`.
   - Intercepted the resulting GET request:
     ```http
     GET /my-account?id=wiener HTTP/1.1
     Host: <lab-id>.web-security-academy.net
     Cookie: session=<session_token>
     ```
   - Noted that account data is retrieved based on the query parameter `id=wiener`.

2. **Parameter Tampering via Burp Repeater:**
   - Sent the `GET /my-account?id=wiener` request to **Burp Repeater**.
   - Modified the `id` parameter from `wiener` to target user `carlos`:
     ```http
     GET /my-account?id=carlos HTTP/1.1
     Host: <lab-id>.web-security-academy.net
     Cookie: session=<session_token>
     ```
   - Dispatched the modified request while keeping the session cookie belonging to `wiener`.

3. **Data Extraction & Verification:**
   - Inspected the HTTP response: the server returned `HTTP/1.1 200 OK`.
   - The response body rendered the account details and API key belonging to `carlos`:
     ```html
     <div>Your API Key is: <span id="apikey">...</span></div>
     ```
   - Copied the exposed API key and submitted it via the lab submission interface.

### 3.4 Actual Observed Result
The application performed no server-side verification to confirm whether the authenticated user associated with the session cookie had permission to access the account profile designated by the `id` parameter. The sensitive API key of user `carlos` was exposed, and the lab was successfully completed.

### 3.5 Security Impact
- **Horizontal Privilege Escalation (BOLA/IDOR):** An attacker can view or tamper with private data belonging to any other user simply by changing an identifier.
- **Confidentiality Breach:** Exposure of sensitive authentication tokens, API keys, or personal identifiable information (PII).

### 3.6 Mitigation
- **Server-Side Access Control Checks:** Validate that the authenticated session owner is strictly authorized to access the requested object before rendering data.
- **Indirect Session References:** Avoid passing user identifiers in client parameters; retrieve the user identity directly from the validated server-side session object (e.g., `currentUser = session.getUser()`).
- **Cryptographically Secure Identifiers:** When reference passing is necessary, utilize non-sequential, cryptographically random identifiers (e.g., UUIDv4) in conjunction with strict authorization policies.

---

## 4. Practical 3: Session Security and Cookie Handling Assessment

### 4.1 Objective
Assess session management controls, including session cookie attribute configuration (`HttpOnly`, `Secure`, `SameSite`), session state handling across the authentication lifecycle, and server-side token invalidation upon user logout.

### 4.2 Tool and Lab Used
- **Primary Tool:** Burp Suite Community Edition (Proxy, Repeater, Cookie Inspector), Browser Developer Tools
- **Testing Scope:** PortSwigger Web Security Academy session management testbed

### 4.3 Steps Performed
1. **Cookie Security Attribute Inspection:**
   - Intercepted the initial authentication exchange and inspected the `Set-Cookie` header issued by the application:
     ```http
     Set-Cookie: session=<token>; Path=/; Secure; HttpOnly; SameSite=None
     ```
   - Evaluated the presence and configuration of essential defense-in-depth flags:
     - `HttpOnly`: Assessed protection against client-side script access (`document.cookie`).
     - `Secure`: Verified restriction of cookie transmission exclusively to encrypted TLS channels.
     - `SameSite`: Examined the cross-site request context behavior.

2. **Session Token Capture & Baseline Request:**
   - Captured the active session cookie value associated with an authenticated session.
   - Sent a request to a protected endpoint (`GET /my-account`) to confirm that the session cookie was valid and returned `200 OK`.

3. **Logout Workflow Analysis:**
   - Triggered the application logout mechanism via the web interface.
   - Intercepted the logout request (`POST /logout` or `GET /logout`) and analyzed the server response headers.
   - Observed whether the server instructed the client browser to overwrite or clear the cookie (e.g., `Set-Cookie: session=; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:00 GMT`).

4. **Post-Logout Session Replay Testing:**
   - Loaded the previously captured authenticated request (`GET /my-account`) into **Burp Repeater**.
   - Retained the old session cookie that was active prior to logout.
   - Re-sent the request to determine if the server still accepted the old token after the logout event had taken place.
   - Inspected response status codes and body content to verify whether access was denied or redirected to `/login`.

### 4.4 Actual Observed Result
- **Cookie Attributes:** Security attributes (`HttpOnly`, `Secure`) were inspected and documented.
- **Logout Behavior:** The application issued client-side cookie clearing instructions upon logout.
- **Replay Assessment:** When replaying the pre-logout session token in Repeater, the server rejected the token or returned an unauthenticated response state (`302 Redirect` to `/login`).
- **Conclusion:** The assessment did **not** conclusively establish an active session-invalidation vulnerability. The tested target did not exhibit improper persistent session reuse under the tested parameters.

### 4.5 Security Impact (Assessed Threat Vector)
- If an application only clears cookies on the client browser without revoking the token in the backend session store, an attacker who intercepts or steals the token can continue to impersonate the user indefinitely until the token reaches absolute timeout.

### 4.6 Mitigation
- **Server-Side Session Revocation:** Ensure the server invalidates and destroys the session state in the active session store (e.g., Redis, database) immediately upon receiving a logout request.
- **Comprehensive Cookie Clearance:** Issue a `Set-Cookie` response header with `Max-Age=0` and an expired timestamp across all relevant domains and paths.
- **Session Re-Generation:** Re-issue new session identifiers upon privilege changes (e.g., login, role escalation) to defend against session fixation attacks.
- **Timeouts:** Enforce both aggressive idle session timeouts and hard absolute session expiration limits.
