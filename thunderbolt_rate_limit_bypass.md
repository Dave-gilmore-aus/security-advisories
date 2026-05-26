# Security Advisory: Rate Limiting Bypass via IP Resolution Failure in Thunderbolt

**Date:** May 26, 2026  
**Author:** Dave-gilmore-aus  
**Software:** Thunderbolt  
**Affected Versions:** 1.0.0  
**Fixed Version:** Unpatched / None  
**Status:** Unresponsive Maintainer (30+ days since private disclosure)  

---

## Executive Summary
The Thunderbolt application features a rate-limiting mechanism intended to protect sensitive endpoints from abuse. However, this defense is completely bypassed because the backend fails to resolve the client's IP address. This flaw exposes critical endpoints—such as authentication and "Waitlist" sign-ups—to automated brute-force, account enumeration, and Denial of Service (DoS) attacks.

## Vulnerability Details
- **Vulnerability Type:** Improper Control of Generation Rate for External Resources (CWE-770) / Rate Limit Bypass
- **Component/File:** Better Auth Middleware Configuration
- **Impact:** Medium/High — Allows for infinite request execution against sensitive endpoints.

### Technical Description
The integrated `Better Auth` middleware is configured to throttle excessive incoming traffic. However, during execution, the backend fails to correctly identify or parse the origin IP from incoming request headers (such as `X-Forwarded-For`). 

Because the client IP evaluates to an empty or unresolvable state, the backend falls back to a warning state and logs:  
`WARN Rate limiting skipped: could not determine client IP address.`

Consequently, the rate limiter treats every incoming request as untracked, completely neutralizing the intended 10-request threshold and allowing continuous, unrestricted access to the endpoint.

## Proof of Concept (PoC)
> ⚠️ **Disclaimer:** This PoC is intended exclusively for educational and defensive security validation.

To reproduce the rate-limit bypass:

1. Open the backend terminal to monitor live server logs.
2. Execute a rapid loop of automated requests to the registration/join endpoint using the following script:

```bash
for i in {1..20}; do 
  curl -i http://localhost:8000/api/auth/join
done

Observed Result: The server continuously responds with 200 OK or 400 Bad Request (depending on payload validity) across all 20+ attempts. It completely fails to trigger an expected 429 Too Many Requests HTTP status code.

Log Verification: Check the backend stdout logs to see the Rate limiting skipped warning printing sequentially for every triggered request.

Impact
Without functional rate limiting, the application faces several severe security risks:

Account Enumeration: Attackers can rapidly probe endpoints to identify valid user email addresses.

Resource Exhaustion & Financial Impact: Flooding endpoints allows attackers to saturate connected databases or exhaust limits on third-party integrations (e.g., Resend), leading to operational downtime or inflated api-usage costs.

Brute-Force Vulnerability: Bad actors can attempt to guess user passwords or One-Time Passwords (OTPs) without facing an IP ban or throttling.

Timeline
April 22, 2026: Initial private disclosure initiated via GitHub Security Advisory to the Thunderbolt maintainers.

May 26, 2026: 34 days elapsed with no response or acknowledgment from the maintainers. Public disclosure published to establish tracking and user awareness.
