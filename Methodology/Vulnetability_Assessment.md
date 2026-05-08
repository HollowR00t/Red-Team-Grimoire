# Vulnerability Assessment and Triage

## Description
Finding a vulnerability is only half the job. The other half is assessing its actual risk to the business. Not all RCEs are critical if they are in an isolated environment, and not all XSSs are low-riskif they allow for the theft of administrator sessions.

To professionally assess vulnerabilities, we rely on industry standards and the context of the attack.

## 1. The Standard: CVSS (Common Vulnerability Scoring System)
This is the universal system for scoring vulnerabilities (currently at v3.1/v4.0). It is based on three groups of metrics:

- **Baseline:** Intrinsic characteristics of the vulnerability (Attack vector, Complexity, Privileges required, User interaction).

- **Temporal:** Factor that change over time (Is there a public exploit? Is a patch available?).

- **Environment:** Modifiers based on the client's specific environment.

*Tool:* Use the official FIRST calculator (https://www.first.org/cvss/calculator/3.1) to obtain the numerical vector (0.0 to 10.0).

## 2. Risk Matrix: Impact vs Probability
At a practical and consulting level, risk is defined as: `Risk = Probability x Impact`.

- **Probability:** How easy is it to exploit? Does it require credentials? Is it exposed to the internet or does it require internal access?

- **Impact:** What happens if it is exploited? Evaluate the damage to the CIA triad (Confidentiality, Integrity, Availability).

## 3. The Red Team's Golden Rule: "Show, Don't Tell"
A theoretical vulnerability carries less weight than a proven one.

To raise the severity of a medium finding to critical, you should always chain vulnerabilities (Vulnerability Chaining).
*Example:* A Medium SSRF + AWS Metadata Access (High) = RCE / Total Compromise (critical).
