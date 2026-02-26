# OIDC Security Updates

## Changes Made

### 1. ✅ Callback Route Supports GET
**Status:** Already implemented
- The `/axis/oidc` handler uses `request.getParameter()` which works for both GET and POST
- No changes needed

### 2. ✅ State Parameter - IMPLEMENTED
**Changes:**
- Added `PARAM_STATE` and `SESSION_STATE` constants
- Generate random 32-byte state parameter before redirect
- Store state in session
- Validate state on callback (exact match required)
- Clear state from session after validation
- Throws `AuthenticationCredentialsNotFoundException` if state validation fails

### 3. ✅ PKCE (Proof Key for Code Exchange) - IMPLEMENTED
**Changes:**
- Added `SESSION_CODE_VERIFIER` constant
- Generate random 64-byte code verifier before redirect
- Generate SHA-256 code challenge from verifier
- Store code verifier in session
- Send `code_challenge` and `code_challenge_method=S256` to `/authorize`
- Send `code_verifier` to `/token` endpoint
- Clear code verifier from session after use

### 4. ⚠️ Callback URL Consistency
**Action Required:**
- Verify that `axis.security.oidc.url.callback` in `security.properties` matches:
  - ACME app configuration
  - The URL used in authorization requests
  - The URL used in token requests
- Current value: `https://axis.disney.network/oidc`
- Avoid using `*.appspot.com` direct access values

### 5. ✅ CSRF Exception Not Needed
**Status:** No changes required
- Since callback is GET with query parameters, no CSRF exemption needed
- No SameSite cookie changes required

## Implementation Details

### New Methods Added:
- `generateRandomString(int length)` - Generates cryptographically secure random strings
- `generateCodeChallenge(String codeVerifier)` - Creates SHA-256 hash for PKCE

### Modified Methods:
- `generateAuthenticationRedirectURL()` - Now accepts `HttpSession` parameter
- `getToken()` - Now accepts `codeVerifier` parameter
- `handleAuthenticationCallback()` - Added state validation and PKCE code verifier retrieval

### Security Improvements:
1. **State parameter** prevents CSRF attacks
2. **PKCE** prevents authorization code interception attacks
3. Both use cryptographically secure random generation (`SecureRandom`)
4. Session-based storage ensures values are tied to user session

## Configuration Checklist

- [ ] Set `axis.security.oidc.url.discovery` in environment-specific properties
- [ ] Verify `axis.security.oidc.url.callback` matches ACME configuration
- [ ] Ensure ACME app supports PKCE (code_challenge_method=S256)
- [ ] Test full OAuth flow with state and PKCE validation
