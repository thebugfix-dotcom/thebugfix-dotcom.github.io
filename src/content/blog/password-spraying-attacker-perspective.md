---
title: "Password Spraying: How an Attacker Hunts for the Weakest Door"
description: "Password spraying seen from the attacker's side: why spreading attempts thin changes the math, what success actually means, and where the strategy breaks down."
pubDate: "2026-10-22"
---

Most people picture a password attack as a siege: one account, thousands of guesses, until the lock gives. Password spraying turns that picture inside out. The attacker does not ask "what is this employee's password?" The question is "somewhere in this organization, is there at least one account still using a password I can predict?" Instead of many guesses against one identity, a spray tries a handful of likely passwords against many identities. [MITRE ATT&CK catalogs it as a sub-technique of Brute Force](https://attack.mitre.org/techniques/T1110/003/) for exactly that reason: it is brute force with the axes swapped.

That swap is the whole strategy. It turns a cracking problem into a probability problem, and probability favors whoever has the larger population to work with.

## The math of "only one has to be wrong"

From the attacker's chair, a spray is a bet on human averages. No single user has to be careless; the organization just has to be large enough that someone, somewhere, chose a password shaped like the season, the company name, or the policy's bare minimum. Each extra account in scope is another lottery ticket, and the attacker only needs one to win.

This is why the technique scales so well against large organizations and so poorly against any one person. A defender who looks at a single account sees almost nothing. The attacker is not playing at the level of the single account at all.

The same logic shows up in real campaigns. Microsoft reported that an Iranian actor it tracks as Peach Sandstorm [ran password spray activity against thousands of organizations](https://www.microsoft.com/en-us/security/blog/2023/09/14/peach-sandstorm-password-spray-campaigns-enable-intelligence-collection-at-high-value-targets/) starting in early 2023, and that follow-on activity appeared in only a small subset of the environments where it authenticated successfully. Breadth first, depth later, and only where it pays.

## Quiet is a hypothesis, not a guarantee

Spreading attempts thin has an obvious appeal: repeated failures against one account attract attention and trip lockouts, while a few failures per account blend into the background noise of forgotten passwords. [CISA described this "low-and-slow" approach](https://www.cisa.gov/news-events/alerts/2018/03/27/brute-force-attacks-conducted-cyber-actors) years ago in an alert on brute force activity by malicious cyber actors.

But "a little per account" is not the same as "invisible." The attacker's quiet depends on nobody stitching the small signals back together. Identity platforms increasingly do exactly that. Microsoft, for example, says it [monitors password spray patterns across IP addresses and other identifiers](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-risks) across tenants. For the attacker, stealth is an assumption baked into the plan, and every system that correlates across users makes that assumption more expensive.

## A correct password is not a breach

Here is where the attacker's view gets more honest than the headlines. "The spray worked" can mean three very different things:

1. **The password was valid.** The credential checked out against the identity provider.
2. **A session was established.** The attacker actually got signed in, past whatever stood behind the password: a second factor, a device or location requirement, a conditional access rule.
3. **The access was worth something.** The session reached mail, files, permissions or relationships that move the operation forward.

Each step is a filter, and each one discards most of what passed the one before. A valid password that hits a second factor may be a dead end. A full sign-in to an account with no useful access is a footnote. From the attacker's side, the real metric is not how many logins were attempted or even how many passwords matched; it is whether one captured identity can become a foothold.

The distinction matters for anyone reading incident reports, too. Microsoft notes that its Entra password spray detection [fires only when an attacker successfully validates a user's password](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-risks); failed spray attempts alone do not trigger that particular detection. That is not a claim that failures are unobservable elsewhere, only a reminder to pin down exactly which stage an alert describes before deciding what happened.

## The identity is the prize

Why go to the trouble? Because a compromised account is rarely the destination. It is a set of borrowed trust relationships: existing permissions, an inbox people answer, a place in the directory, and activity that looks normal because it comes from a real employee. Actions taken under a legitimate identity tend to blend into the daily routine, and that blending buys the attacker time.

The clearest public example is Microsoft's own. In January 2024 the company disclosed that the Russian state-sponsored actor Midnight Blizzard had [used a password spray attack to compromise a legacy non-production test tenant account and gain a foothold](https://www.microsoft.com/en-us/msrc/blog/2024/01/microsoft-actions-following-attack-by-nation-state-actor-midnight-blizzard), then used that account's permissions to reach a small percentage of corporate email accounts, including senior leadership. The sprayed account itself was unremarkable. What mattered was what it was connected to.

That is the attacker's real goal in a spray: convert a small weakness spread across a large population into one usable point of entry.

## Where the strategy breaks

The model has blind spots, and the attacker has to price every one of them in:

- **Organizations are not one uniform login.** Different services use different sign-in flows, different authentication layers and different policies. An account that exists in the directory is not necessarily an account that can sign in everywhere.
- **The password is no longer the last gate.** Phishing-resistant factors and context-aware access policies sit after the password check, which is exactly where a spray's value is supposed to materialize.
- **Correlation erases the advantage.** The technique lives in the gap between the local view (each account saw a little) and the system view (the organization saw a pattern). The more an environment looks at the system view, the less a spray buys.
- **Predictable passwords are a shrinking pool.** Guidance such as [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html) pushes verifiers to block commonly used and compromised passwords, which goes straight at the attacker's assumption that someone, somewhere, picked the obvious one.

Each of these turns "spray and hope" into a longer chain of uncertain links, and attackers, like everyone else, prefer short chains.

## The gap is the story

Password spraying is ultimately a story about perspective. On any single account, almost nothing happens. Across the whole organization, a pattern emerges. The attacker is betting that nobody looks at the whole, and that somewhere in the crowd a guessable password, a missing second factor and a well-connected account happen to line up.

The difference between a guessed password, a successful sign-in and a real foothold is the entire game.

*This post discusses attacker logic at a strategic level only. Hands-on testing belongs in a personal lab or on systems you are explicitly authorized to test.*
