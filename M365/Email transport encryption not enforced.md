# Email Transport Encryption Not Enforced

**Severity:** Medium  
**Type:** External M365 Penetration Test  

---

## The Scope

External penetration test against a client's internet-facing infrastructure and Microsoft 365 tenant.

## How Did I Discover It?

After wrapping up the email spoofing finding on the same engagement, I looked at the transport layer  how emails are protected while moving between mail servers.

STARTTLS is what upgrades SMTP from plaintext to encrypted, but it's opportunistic by default. A man-in-the-middle can strip the `250-STARTTLS` capability from the EHLO response, and the sending server silently falls back to plaintext. The fix is MTA-STS  a policy that tells senders "require TLS or don't deliver."

```
$ dig +short TXT _mta-sts.[client domain]
(no output)
```

No MTA-STS record published. I confirmed with an SMTP probe using `swaks --no-tls` the connection completed in plaintext. The relay also advertised `AUTH PLAIN` and `AUTH LOGIN` before TLS negotiation, so credentials over a downgraded session would be exposed in base64 to anyone watching.

## What Did I Learn?

- **DMARC and MTA-STS solve different problems.** DMARC protects the sender's identity. MTA-STS protects the channel. Having one doesn't cover the other.
- **STARTTLS without enforcement is security theatre.** It encrypts when nobody's attacking and fails silently when someone is.
- **Always check both layers.** After finding the spoofing bug it would've been easy to move on, but email security has multiple independent controls and each one deserves its own check.
