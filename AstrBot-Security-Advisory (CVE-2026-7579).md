Hardcoded Credentials & Timing Attack in AstrBot Dashboard (CVE-2026-7579)

Link to CVE: https://vuldb.com/vuln/360420

Disclosed to Dev Date: Feb 14, 2026

Researcher: David Gilmore

Severity: CRITICAL (Estimated CVSS: 9.8)

Vulnerability Types: * CWE-798: Use of Hard-coded Credentials

CWE-208: Observable Timing Differential (Timing Attack)

1. Summary
The AstrBot AI agent platform (v4.16.0) contains a critical authentication bypass vulnerability in its web dashboard. The system utilizes hardcoded default credentials that are not forcibly disabled or changed upon deployment. Furthermore, the authentication comparison logic is susceptible to a timing attack, allowing for a potential brute-force of the dashboard password even if it has been changed from the default.

2. Technical Analysis
A. Hardcoded Backdoor (CWE-798)
The logic in astrbot/dashboard/routes/auth.py contains a static check for a default username and a hashed password. While a warning is logged, the login is still permitted.

Vulnerable Code Sink:

Python
# astrbot/dashboard/routes/auth.py (Lines 33-37)
if (username == "astrbot" and password == "77b90590a8945a7d36c963981a307dc9" and not DEMO_MODE):
    change_pwd_hint = True
    logger.warning("为了保证安全，请尽快修改默认密码。") # Warning: Modify default password as soon as possible.
B. Timing Attack (CWE-208)
The use of the standard equality operator (==) for password validation allows an attacker to measure the time taken for the comparison. Since == returns as soon as it finds a mismatch, an attacker can determine the correct password character-by-character.

3. Proof of Concept (PoC)
Direct Access: Navigate to the AstrBot dashboard login.

Default Credentials: Enter astrbot as the username and the password corresponding to the hash 77b90590a8945a7d36c963981a307dc9 (Note: The hash itself is the static credential in this context).

Result: Full administrative access to the bot's configuration, logs, and integrated LLM API keys.

4. Impact
Full System Compromise: An attacker can reconfigure the bot, steal API keys (OpenAI, Claude, etc.), and intercept user conversations.

Remote Execution: If the bot has "Shell" or "MCP" tools enabled, an attacker with dashboard access can execute arbitrary commands on the host system.

5. Recommended Remediation
Mandatory Initial Setup: Force users to set a unique password on the first run; remove all hardcoded fallbacks from the source code.

Constant-Time Comparison: Implement secrets.compare_digest() for all credential checks.

Secure Hashing: Replace simple string comparisons with a modern hashing algorithm like Argon2 or bcrypt.
