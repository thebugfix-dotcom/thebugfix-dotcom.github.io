---
title: 'Password Spraying: What It Is and How to Defend Against It'
description: 'A defender-focused guide to password spraying: how the pattern looks, how to detect it in your logs, and the controls that stop it.'
pubDate: 'Oct 22 2026'
---

<svg role="img" aria-label="One password tried against many locked doors" style="width:100%;height:auto" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 500"><rect width="1200" height="500" fill="#06213a"/><g fill="#0d4a6b" stroke="#2bb7c4" stroke-width="2"><rect x="150" y="150" width="60" height="200"/><rect x="250" y="150" width="60" height="200"/><rect x="350" y="150" width="60" height="200"/><rect x="450" y="150" width="60" height="200"/><rect x="550" y="150" width="60" height="200"/><rect x="650" y="150" width="60" height="200"/><rect x="750" y="150" width="60" height="200"/><rect x="850" y="150" width="60" height="200"/><rect x="950" y="150" width="60" height="200"/><rect x="1050" y="150" width="60" height="200" fill="#6b4a0d" stroke="#f5a623"/></g><path d="M80 250 L1140 215 L1140 285 Z" fill="#2bb7c4" opacity=".25"/><g fill="#2bb7c4"><circle cx="50" cy="250" r="22"/><rect x="60" y="244" width="50" height="12"/></g><text x="600" y="440" font-family="sans-serif" font-size="34" fill="#cfe9ee" text-anchor="middle">One password, many accounts</text></svg>

Password spraying is one of the most common ways accounts get taken over, and one of the easiest to defend against once you know what it looks like. This post explains the idea at a high level, then focuses on what defenders should watch for and which controls actually work.

## What is password spraying?

Most people know the classic brute force attack: many password guesses against a single account. It is loud, and it trips account lockouts quickly.

Password spraying flips the pattern. An attacker takes a very small number of common passwords and tries each one against many different accounts, slowly. Because each account sees only one or two failed sign-ins, normal per-account lockout rules often never fire.

<svg role="img" aria-label="Brute force compared with password spraying" style="width:100%;height:auto" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 560"><rect width="1200" height="560" fill="#eef7f8"/><g font-family="sans-serif" text-anchor="middle"><rect x="20" y="20" width="570" height="80" rx="14" fill="#14606e"/><text x="305" y="73" font-size="40" font-weight="700" fill="#fff">Brute force</text><rect x="610" y="20" width="570" height="80" rx="14" fill="#14606e"/><text x="895" y="73" font-size="40" font-weight="700" fill="#fff">Password spray</text><text x="305" y="150" font-size="26" fill="#0d2b45">Many passwords, one account</text><text x="895" y="150" font-size="26" fill="#0d2b45">One password, many accounts</text><g fill="#0d3b5c"><rect x="265" y="250" width="80" height="70" rx="8"/><rect x="760" y="210" width="60" height="50" rx="6"/><rect x="860" y="210" width="60" height="50" rx="6"/><rect x="960" y="210" width="60" height="50" rx="6"/><rect x="760" y="300" width="60" height="50" rx="6"/><rect x="860" y="300" width="60" height="50" rx="6"/><rect x="960" y="300" width="60" height="50" rx="6"/><rect x="760" y="390" width="60" height="50" rx="6"/><rect x="860" y="390" width="60" height="50" rx="6"/><rect x="960" y="390" width="60" height="50" rx="6"/></g><g stroke="#f5a623" stroke-width="5"><path d="M120 200 L255 270"/><path d="M120 285 L255 285"/><path d="M120 380 L255 305"/><path d="M490 200 L355 270"/><path d="M490 285 L355 285"/><path d="M490 380 L355 305"/></g><g stroke="#14606e" stroke-width="5" stroke-dasharray="10 8"><path d="M640 305 L750 235"/><path d="M640 305 L750 325"/><path d="M640 305 L750 415"/></g><circle cx="620" cy="305" r="22" fill="#14606e"/><text x="305" y="500" font-size="24" fill="#0d2b45">Trips per-account lockout fast</text><text x="895" y="500" font-size="24" fill="#0d2b45">Stays under per-account lockout</text></g></svg>

The attack works because, in any large group of users, a few will always pick weak or predictable passwords. The attacker does not need to be clever. They only need volume across accounts and patience.

## Why it is hard to notice

- **Low per-account volume.** One failure per account looks like a typo.
- **Spread over time and sources.** Attempts may be spaced out and come from many IP addresses.
- **Targets exposed sign-in points.** Email, VPN, single sign-on portals and remote access services are the usual targets, especially those without multi-factor authentication.

## How defenders can detect it

The signal is not in any single account. It is in the pattern across accounts. Useful things to look for:

1. **Many accounts, few failures each.** A source (or set of related sources) producing failed sign-ins across a large number of distinct usernames.
2. **Alphabetical or list-like username order.** Failures that walk through usernames in a predictable order.
3. **Failures followed by a success.** A burst of failed attempts across accounts, then a successful sign-in on one of them from the same source.
4. **Unusual sign-in sources.** Successful logins from new countries, hosting providers, anonymizing services or user agents your organization does not normally see.
5. **Spikes at the identity provider.** A jump in total failed sign-ins on your single sign-on or email service, even if no individual account is locked.

Practical steps:

- Collect authentication logs centrally (identity provider, VPN, email, remote access) and keep them long enough to look back several weeks.
- Build alerts on distinct usernames failing per source over a time window, not just failures per account.
- Alert on a success that follows a pattern of failures across accounts.
- Review sign-in risk reports your identity provider already offers, and turn on its built-in detections.

## How to prevent it

No single control is enough, but together they make spraying close to useless.

<svg role="img" aria-label="Layered defenses against password attacks" style="width:100%;height:auto" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 560"><rect width="1200" height="560" fill="#f7fbfc"/><g font-family="sans-serif" text-anchor="middle"><circle cx="600" cy="280" r="250" fill="#f5a623"/><circle cx="600" cy="280" r="200" fill="#0d3b5c"/><circle cx="600" cy="280" r="150" fill="#14909c"/><circle cx="600" cy="280" r="100" fill="#a8e4ea"/><circle cx="600" cy="280" r="40" fill="#0d2b45"/><text x="600" y="75" font-size="26" font-weight="700" fill="#0d2b45">MFA</text><text x="600" y="125" font-size="22" fill="#fff">Banned passwords</text><text x="600" y="175" font-size="22" fill="#fff">Smart lockout</text><text x="600" y="225" font-size="20" fill="#0d2b45">Sign-in monitoring</text><text x="600" y="290" font-size="18" fill="#fff">Account</text></g></svg>

### 1. Require multi-factor authentication

MFA is the highest impact control. Even when a guessed password is correct, the attacker still cannot sign in. Prefer phishing-resistant methods (passkeys or security keys) and authenticator apps over SMS where you can. Make sure legacy protocols that bypass MFA are disabled.

### 2. Ban weak and compromised passwords

Spraying depends on common passwords. Block them at the source:

- Reject passwords found on common-password and breached-password lists.
- Block obvious patterns such as the company name, season plus year, or "Welcome1".
- Favor length over complexity rules. Long passphrases beat short passwords with forced symbols.

### 3. Use smart lockout instead of simple lockout

Simple lockout after N failures per account is easy to evade with spraying, and it also lets attackers lock out real users on purpose. Smart or adaptive lockout looks at the source and context of attempts, tracks familiar locations and devices, and slows down suspicious traffic without locking out legitimate users. Combine it with rate limiting per source and per username.

### 4. Reduce the attack surface

- Put remote access and admin portals behind MFA and, where possible, a VPN or zero trust gateway.
- Disable accounts that are no longer used, and remove default or shared accounts.
- Avoid exposing username lists. Make sign-in error messages generic so they do not reveal which usernames exist.

### 5. Monitor and respond

- Alert on the patterns described above and have a clear runbook.
- If you detect spraying, force password resets and revoke sessions for any account that had a successful sign-in from the suspicious source, then block or throttle the source.
- Review accounts with weak or reused passwords proactively, using password audit tooling approved by your organization.

## A quick checklist

- [ ] MFA enforced for all users, especially email, VPN and admin accounts
- [ ] Legacy authentication that skips MFA is disabled
- [ ] Banned password list and a sensible minimum length are in place
- [ ] Smart lockout and rate limiting are enabled
- [ ] Logs from all sign-in points are centralized
- [ ] Alerts exist for many accounts failing from one source
- [ ] An incident response step exists for suspected account compromise

## The takeaway

Password spraying succeeds by staying under the radar of per-account defenses. Defend at the level of the pattern: stop weak passwords from existing, make a correct password alone worthless with MFA, and watch for failures across accounts, not just within one.
