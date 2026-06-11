# MailPlus Server

- **DNS provider** (e.g. namecheap.com): setup DNS records
  - **A** record: `mail` -> your public IPv4
  - **MX** record: `@` -> `mail.awesome.com` (priority 10)
  - Additional **TXT** records are required for delivery, see the following section
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
  - Allow relevant ports on firewall (i.e. 25, 465, 993)
  - Disable IMAP/POP3/POP3 SSL/TLS
  - Disable SMTP-STARTTLS
  - Enable user for having the mail account
  - Control Panel -> User & Group
  - Ensure MailPlus Server app is enabled for the user

## Delivery

{% hint style="warning" %}
Reverse DNS (PTR) should be aligned to the forward DNS.
If PTR is not configurable for your ISP, consider using relay servers instead.
Use following commands to double check forward and reverse DNS.
```bash
dig +short -x <YOUR_PUBLIC_IP>
dig +short mail.awesome.com A
```
{% endhint %}

- **SPF**
  - **DNS provider** (e.g. namecheap.com): set DNS record **TXT** at `@`: `v=spf1 mx a:mail.awesome.com -all`
- **DMARC**
  - **DNS provider** (e.g. namecheap.com): set DNS record **TXT** at `_dmarc`: `v=DMARC1; p=none; rua=mailto:postmaster@awesome.com; adkim=s; aspf=s; fo=1`
  - (Optional) Move to `p=quarantine` after stable SPF/DKIM pass
  - (Optional) Move to `p=reject` when confident
- **DKIM**
  - **MailPlus Server** -> Domain -> edit `awesome.com` -> Advanced (DKIM section)
    - Enable DKIM signing for the domain
    - DKIM selector prefix: default
    - Generate key pair (prefer `2048` if available)
  - **DNS provider** (e.g. namecheap.com): set DNS record **TXT** at `default._domainkey`: `v=DKIM1; k=rsa; p=<paste here the public key generated>`

### Relay Server (Azure)

- Communication Services -> Create
  - Set name and region, e.g. `acs-awesome`
- Email Communication Services -> Create
  - Set name and same region as before, e.g. `acs-awesome-email`
  - Provision domains -> Add domain -> Custom domain
    - Enter domain name, e.g. `awesome.com`
    - Initiate the verification process
    - **DNS provider** (e.g. namecheap.com): set DNS record **TXT** at `@`: `<domain verification value from azure>`
    - Azure also provides SPF (**TXT** at `@`) and DKIM/DKIM2 (**CNAME** records): add all of them
      
      Note: if you already have an SPF record at `@`, merge the Azure include into it rather than adding a second TXT, e.g. `v=spf1 mx a:mail.awesome.com include:spf.protection.outlook.com -all`
- Entra ID -> App registrations -> New registration
  - Name it, e.g. `acs-smtp-synology`
  - Register
  - Create a secret and note it down
- Communication Services -> Access control (IAM) -> Add role assignment
  - Select role _Communication and Email Service Owner_
  - Select app registration created before, e.g. `acs-smtp-synology`
- Communication Services -> Email -> SMTP Username -> Add SMTP Username
  - Entra Application: e.g. `acs-smtp-synology`
  - Username type: Email
  - Username: e.g. `relay@awesome.com`
- Communication Services -> Email -> Domains -> `awesome.com` -> Email services -> MailFrom Addresses
  - Add your accounts, e.g. `samuel@awesome.com`

    Note: if you are unable to add the MailFrom using the portal, use the following command:
    `az communication email domain sender-username create --email-service-name acs-awesome-email --resource-group rg-awesome --domain-name awesome --sender-username samuel --username samuel --display-name Samuel`

- **MailPlus Server** -> Mail Delivery -> Relay Settings
  - Check _Send emails through relay servers_
  - Server List -> Add
    - Server: `smtp.azurecomm.net` (verify in Azure portal: Communication Services -> Settings -> SMTP)
    - Port: 587
    - Username: SMTP username created above, e.g. `relay@awesome.com`
    - Password: client secret

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

- Router access for port forwarding
- Dedicated mail host name (recommended): `mail.awesome.com`
- (For selfhosted delivery)
  - ISP allows outbound `25/tcp`
  - PTR (reverse DNS) can be configured for your public IP

## References
- [Synology MailPlus License Pack](https://www.synology.com/en-global/products/MailPlus_License)
- [Synology: How to Set Up Email Using Your Own Domain Name](https://mariushosting.com/synology-how-to-set-up-email-using-your-own-domain-name/)
- [PTR Record (Italian)](https://managedserver.it/cose-un-record-ptr-e-perche-e-essenziale-in-un-mail-server/)