# Security Advisory: Missing HttpOnly Attribute on Authentication Session Cookie in Thunderbolt

**Date:** May 26, 2026  
**Author:** Dave-gilmore-aus  
**Software:** Thunderbolt  
**Affected Versions:** 1.0.0  
**Fixed Version:** Unpatched / None  
**Status:** Unresponsive Maintainer (30+ days since private disclosure)

---

## Executive Summary

The primary authentication session cookie (`__session`) used by Thunderbolt is not configured with the `HttpOnly` security attribute. This omission allows the sensitive JSON Web Token (JWT) contained within the cookie to be read by client-side JavaScript. Consequently, the application faces a significantly elevated risk of full account takeover if an attacker successfully exploits any Cross-Site Scripting (XSS) vulnerability on the platform.

## Vulnerability Details

- **Vulnerability Type:** Sensitive Cookie Without `HttpOnly` Flag (CWE-1004)
- **Component/File:** Session Management / Cookie Issuance Logic
- **Impact:** High — Exposes sensitive authentication tokens to client-side scripts, neutralizing a critical layer of defense against session theft.

### Technical Description

During cookie generation, the application sets the `__session` cookie containing the user's active authentication token but fails to append the `HttpOnly` directive in the HTTP response headers.

By default, modern web browsers restrict scripts from accessing cookies protected by the `HttpOnly` flag. Because this flag is missing in Thunderbolt, the cookie is completely accessible via the browser's `document.cookie` API, allowing untrusted client-side scripts to interact with, read, and potentially exfiltrate the active user session.

## Proof of Concept (PoC)

> ⚠️ **Disclaimer:** This PoC is intended exclusively for educational and defensive security validation.

To verify the missing `HttpOnly` attribute:

1. Open the Thunderbolt application in any modern web browser and log into an account.
2. Open the browser's Developer Tools (`F12`) and select the **Console** tab.
3. Type the following command and hit Enter:

```javascript
console.log(document.cookie);
```

### Observed Result

The console prints the full contents of the `__session` cookie, revealing the sensitive JWT.

### Alternative Verification

Navigate to the **Application** (or **Storage**) tab, select **Cookies** from the left sidebar, and inspect the `__session` row. The `HttpOnly` checkbox or column will be empty/unchecked.

## Impact

While missing the `HttpOnly` flag does not allow an attacker to breach the app on its own, it acts as a major risk multiplier:

- **Session Theft via XSS:** If a Cross-Site Scripting (XSS) vulnerability exists anywhere within the application domain, an attacker can execute a script to silently steal the `__session` token.
- **Account Takeover:** The stolen JWT can be imported into an attacker's browser session, allowing them to impersonate the victim, potentially bypass multi-factor protections (depending on implementation), and gain unauthorized access to the account.

## Timeline

- **march 10, 2026:** Initial email to maintainers to advise of vulnerability.
- **April 22, 2026:** Disclosure initiated via GitHub Security Advisory to the Thunderbolt maintainers.
- **May 26, 2026:** 77 days elapsed with no response or acknowledgment from the maintainers. Public disclosure published to establish tracking and user awareness.
