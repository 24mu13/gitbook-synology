# SSL/TLS

The following guides assume you are using **namecheap.com** as your domain registrar, 
but the process should be similar for other registrars.

## Activate/renew certificate

First you need to get a CSR (Certificate Signing Request) from DSM, e.g. for renewal:
- Control Panel → Security → Certificate
- Click on **Renew**
- Copy the CSR (Certificate Signing Request)

See: [Domain & CSR code setup for Synology NAS](https://www.namecheap.com/support/knowledgebase/article.aspx/9634/2290/domain-csr-code-setup-for-synology-nas/)

### Registrar (i.e. namecheap.com)

- SSL Certificates → Manage
- Click on **Activate**/**Reissue**
- Paste the CSR got from DSM (above)
- Choose CNAME record add
- Get the record name and value
- Domains List → Domain → Advanced DNS
- Add CNAME record
  - *Host*: copy ID here without the suffix ".<domain>"
  - *IP Address*: copy host here replacing ".comodoca.com" with ".sectigo.com"

See: [Steps to get the SSL certificate activated](https://www.namecheap.com/support/knowledgebase/article.aspx/794/67/how-do-i-activate-an-ssl-certificate/)

### DSM

- Control Panel → Security → Certificates
- Select **Add**/**Replace**
- Import certificate
  - From **archive.zip** downloaded when generating CSR:
    - Private Key: *server.key*
  - From **<domain>.zip** downloaded from namecheap:
    - Certificate: *<domain>.crt*
    - Intermediate Certificate: *<domain>.ca-bundle*

See: - [Installing an SSL certificate on Synology NAS](https://www.namecheap.com/support/knowledgebase/article.aspx/9635/33/installing-an-ssl-certificate-on-synology-nas/)