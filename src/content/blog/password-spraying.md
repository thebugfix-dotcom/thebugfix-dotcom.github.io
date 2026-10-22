---
title: "Password spraying: how it works, how to catch it, how to test for it safely"
description: "A pentester's view of password spraying: why it slips past lockout, what it looks like in logs, and which controls actually stop it."
pubDate: "2026-10-22"
heroImage: "./assets/password-spraying-cover.png"
---

Password spraying is boring, which is why it works. One password, or a very short list, tried against many accounts. No clever exploit, no malware. Just the odds that somewhere in a few thousand accounts, someone picked `Season2026!`.

I run into it from both sides: as the person who tests for it in authorized engagements, and as the person who explains to defenders why their lockout policy did not notice it. This post covers how it works at a conceptual level, what it looks like in logs, and what to do about it.

## What it is

MITRE ATT&CK tracks it as [T1110.003, Brute Force: Password Spraying](https://attack.mitre.org/techniques/T1110/003/). The description is short: an adversary uses a single password, or a small list of commonly used passwords, against many accounts, "to avoid account lockouts that would normally occur when brute forcing a single account with many passwords."

That sentence is the whole idea. Classic brute force goes deep on one account. Spraying goes wide, so that each account sees only one or two failures.

OWASP's [Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) draws the same line: brute force tests many passwords against one account, password spraying tests one weak password against many accounts.

It is related to, but not the same as, credential stuffing (T1110.004). Stuffing replays real username and password pairs leaked elsewhere. Spraying guesses.

## Why lockout does not save you

A per-account lockout counts failures per account. A spray produces one failure per account. Each counter stays near zero, so nothing locks and nothing alerts.

Attackers also slow down on purpose. MITRE notes that adversaries "may deliberately throttle password spraying attempts to avoid triggering security alerting," and that they may use LDAP and Kerberos authentication rather than SMB, which is less likely to produce the very visible Windows logon failure event 4625. Its procedure examples include a campaign that used a single sign-in attempt per account per 24 hours (Quad7 Activity), and APT28 tooling that made roughly four attempts per hour per targeted account over days or weeks.

Targets are usually the same few places: externally facing email such as OWA and Microsoft 365, VPN and remote access portals, SSO and other cloud apps behind federated authentication, and management services such as SSH, RDP and SMB.

## What it looks like in logs

The signal is not "many failures for one user." It is the shape across users:

- Many different accounts failing in a short window.
- The same source IP, or a small set of IPs (or a large rotating set, which is harder), behind those failures.
- Failures that cluster around the same client or user agent.
- Then one success in the middle of the failures. That success is the one that matters.

In Microsoft Entra ID sign-in logs, a wrong password shows up as error [50126](https://learn.microsoft.com/en-us/entra/identity-platform/reference-error-codes) (invalid username or password), and 50053 means the account is locked. A burst of 50126 across many users from one place is the pattern to hunt for. Microsoft's own docs say to expect some 50126 noise from ordinary typos, so you need a baseline, not a single alert on the code.

ATT&CK's detection strategy for this technique (DET0487) is the same idea stated generally: a high volume of authentication failures using one password, or a small set, across many accounts in a time window. It lists variants for SSH and PAM, enterprise SSO and SaaS, mailboxes, containers and network devices.

One caveat from Microsoft: the Entra ID Protection "password spray" risk detection [only triggers when an attacker successfully validates a user's password](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-risks). Unsuccessful attempts against your tenant are not flagged by it. If you only wait for that detection, you find out after the first win. Build your own detection on failures too.

## What actually helps

In rough order of value:

**1. Multi-factor authentication.** OWASP calls MFA "by far the best defense against the majority of password-related attacks," and ATT&CK lists it as mitigation M1036's neighbor, M1032. A guessed password with a second factor behind it is not an account takeover. It is still worth alerting on, because a correct password followed by an MFA prompt the user did not start tells you the password is known.

**2. Block weak passwords at the source.** NIST SP 800-63B says verifiers must compare new passwords against a blocklist of commonly used, expected or compromised values, and must not impose composition rules or periodic rotation. Microsoft Entra Password Protection does this with a global banned list (not published, so attackers cannot read it) plus a custom list of up to 1,000 terms. Put your company name, product names and local terms in the custom list. Those are the guesses a targeted spray tries first.

**3. Smart lockout, not just lockout.** Entra smart lockout is on by default. Default settings are a threshold of 10 failed attempts and a 60 second lockout in Azure public tenants, and it keeps separate counters for familiar and unfamiliar locations, so a real user is less likely to be locked out by an attacker's guesses. If you sync with on-premises AD, Microsoft recommends the cloud threshold be lower than the AD DS threshold, and the cloud lockout duration longer, so a spray against the cloud does not lock accounts on-prem. Aggressive hard lockout also hands the attacker a denial of service: they can lock out everyone on purpose. ATT&CK M1036 makes the same point.

**4. Conditional access and network controls.** Block legacy authentication, limit sign-ins to compliant devices or expected locations where you can, and treat anonymizing proxies as risky. Spraying from a residential proxy network will not trip IP reputation, so do not rely on that alone.

**5. Generic errors and rate limits.** OWASP's guidance is to return the same response whether or not the user exists, and to rate limit by more than one dimension. User enumeration makes spraying more efficient, so do not give it away.

## Testing for it in an authorized engagement

This part is for people who have a signed scope. If you do not have written authorization for the exact accounts and services, do not test.

My rules, and the ones I would ask any tester to follow:

- **Get the lockout policy before you start.** Ask for the threshold, the observation window and the duration. Do not discover it by locking someone out.
- **Stay well under the threshold.** One attempt per account per window, with a margin, is the usual approach. If the policy is unknown, assume it is stricter than you think.
- **Agree the account list and the time windows in writing.** Avoid business-critical accounts and shared service accounts unless the client wants them in.
- **Tell the blue team the plan, or agree on a deliberate stealth test.** Both are valid. Surprise tests tell you more about detection, but they need explicit sign-off.
- **Log everything you do.** Time, source IP, accounts attempted, password tried. If something locks, you can show exactly what happened.
- **Stop at proof.** One valid credential proves the finding. Report it, and do not read mailboxes or move laterally unless the rules of engagement say so.
- **Report detection, not just success.** The useful result is often "we sprayed N accounts and nothing alerted," or "alert fired after M minutes." Say which.

The deliverable that helps the client most is the combination: which weak passwords existed, and whether anyone noticed.

## Quick checklist for defenders

- MFA on every externally reachable login, no legacy auth.
- Banned password list with your own terms added.
- Smart lockout tuned, and cloud and on-prem thresholds set so one cannot lock the other.
- Detection on failures across accounts per source, not only per account.
- Alert on a success that follows a burst of failures from the same source.
- A baseline for 50126 and the equivalents in your other systems.
- Test it yearly, with a scope and a lockout-aware plan.

## Sources

- MITRE ATT&CK, T1110.003 Password Spraying: https://attack.mitre.org/techniques/T1110/003/
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Microsoft Entra smart lockout: https://learn.microsoft.com/en-us/entra/identity/authentication/howto-password-smart-lockout
- Microsoft Entra Password Protection (banned passwords): https://learn.microsoft.com/en-us/entra/identity/authentication/concept-password-ban-bad
- Microsoft Entra ID Protection risk detections (password spray): https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-risks
- Microsoft Entra authentication error codes: https://learn.microsoft.com/en-us/entra/identity-platform/reference-error-codes
- NIST SP 800-63B-4: https://pages.nist.gov/800-63-4/sp800-63b.html
