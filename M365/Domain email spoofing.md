# Email Spoofing via Unprotected Legacy Domain

**Severity:** Critical  
**Type:** External M365 Penetration Test  

---

## The Scope

External penetration test against a client's internet-facing infrastructure and Microsoft 365 tenant. The engagement covered multiple external IPs and all M365-related services. Phishing and DoS were out of scope.

## How Did I Discover It?

It started with a simple WHOIS lookup on the client's primary domain. The registrant organisation name came back different from the company's current trading name an older name. That caught my attention.

I searched for the old name online and found a legacy company website still live on a third-party hosting platform. The site had a team page listing employee names and email addresses, all under an almost identical domain to the new one. Both domains pointed to the same MX records, so I checked whether they shared the same M365 tenant by querying Microsoft's public OpenID configuration endpoint. The tenant IDs matched same tenant, two domains.

Next, I checked the email authentication records on the legacy domain:

- **SPF:** Published, but without DMARC it's not enforced on the receiving side  
- **DMARC:** Not configured  
- **DKIM:** Not configured  

The primary domain had full protection (DMARC `p=reject`, DKIM active, strict SPF). But the legacy domain had none of it. Because both sit in the same tenant, the legacy domain is a valid sender identity  and it's completely spoofable.

I sent a test email impersonating `admin@[legacy domain]` using a web-based SMTP tool. It landed:
![[Pasted image 20260623011129.png]]
![[Pasted image 20260623011112.png]]

The email hit spam rather than inbox, which means the receiving server's spam filters are doing their job but the email was still **accepted and delivered**, not rejected. With a clean sending IP (not a known open mailer), delivery to inbox is realistic.

## What Did I Learn?

- **Always check for sibling domains.** WHOIS, certificate transparency logs, and tenant ID lookups can reveal domains the client forgot about. A single unprotected domain in a shared tenant undoes all the work on the primary one.
- **DMARC is the enforcement layer.** SPF alone doesn't stop spoofing  without a DMARC policy, receiving servers have no instruction to reject failures. This is a common misconception.
- **Legacy assets are real attack surface.** Old websites, old domains, and old DNS records don't stop being dangerous just because the client stopped using them. Attackers look for exactly this kind of gap.
- **Enumeration wins engagements.** This finding didn't come from a scanner. It came from reading a WHOIS record carefully and following the thread. Recon is where the critical findings live.
