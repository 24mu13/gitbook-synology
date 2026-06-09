# MailPlus Server

- **DNS provider** (e.g. namecheap.com): setup DNS records
  - **A** record: `mail` -> your public IPv4
  - **MX** record: `@` -> `mail.awesome.com` (priority 10)
  - **TXT (SPF)** at `@`:
    ```text
    v=spf1 mx a:mail.awesome.com -all
    ```
  - **TXT (DMARC)** at `_dmarc` (start with monitoring):

    ```text
    v=DMARC1; p=none; rua=mailto:postmaster@awesome.com; adkim=s; aspf=s; fo=1
    ```
  - (Optional) **AAAA** record: `mail` -> your public IPv6 only if your NAS is truly reachable on IPv6

- **Router**: port forwarding to NAS
  - `993/tcp` (IMAP SSL/TLS)
  - `465/tcp` (SMTP SSL/TLS)
  - `25/tcp` (SMTP, server-to-server)
  - `80/tcp` for Let's Encrypt HTTP validation

- Add new certificate (i.e. Let's Encrypt)
  - Control Panel -> Security -> Certificate
  - Add -> Add a new certificate
  - Get a certificate from Let's Encrypt
  - Domain name: `mail.awesome.com`
  - Email: your admin email
  - Assign that certificate to mail-related services

- Install and configure **MailPlus Server**
  - Package Center -> search for MailPlus Server -> Install
  - Allow relevant ports on firewall (i.e. 993)
  - Disable IMAP/POP3/POP3 SSL/TLS
  - Disable SMTP-STARTTLS
  - Enable user for having the mail account
  - Control Panel -> User & Group
  - Ensure MailPlus Server app is enabled for the user

## Deliverability hardening

### DKIM

- **MailPlus Server** -> Domain -> edit `awesome.com` -> Advanced (DKIM section)
- Enable DKIM signing for the domain
- DKIM selector prefix: default
- Generate key pair (prefer `2048` if available)
- **Provider** (e.g. namecheap.com): setup DNS record
  - **TXT (DKIM)** at `default._domainkey`:
    ```text
    v=DKIM1; k=rsa; p=<paste here the public key generated>
    ```

### DMARC

- Start with `p=none`
- Move to `p=quarantine` after stable SPF/DKIM pass
- Move to `p=reject` when confident

### PTR

Reverse DNS (PTR) should be aligned to the forward DNS.

```bash
dig +short -x <YOUR_PUBLIC_IP>
dig +short mail.awesome.com A
```

If PTR cannot be enabled:
- Keep mailbox on DSM
- Use SMTP relay for outbound only
- Keep SPF/DKIM/DMARC configured for that relay

## Client Settings

- Username: `<as DSM user>@awesome.com`
- Password: `<as DSM password>`
- Incoming
  - Server: `mail.awesome.com`
  - Port: 993
  - Type: IMAP SSL/TLS
- Outgoing
  - Server: `mail.awesome.com`
  - Port: 465
  - Type: SMTP SSL/TLS

## Assumptions

- Domain on Namecheap BasicDNS (e.g. `awesome.com`)
- Router access for port forwarding
- Dedicated mail host name (recommended): `mail.awesome.com`
- ISP allows outbound `25/tcp`
- PTR (reverse DNS) can be configured for your public IP

## References
- [Synology MailPlus License Pack](https://www.synology.com/en-global/products/MailPlus_License)
- [Synology: How to Set Up Email Using Your Own Domain Name](https://mariushosting.com/synology-how-to-set-up-email-using-your-own-domain-name/)
- [PTR Record (Italian)](https://managedserver.it/cose-un-record-ptr-e-perche-e-essenziale-in-un-mail-server/)