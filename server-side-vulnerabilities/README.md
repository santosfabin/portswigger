# Server-side vulnerabilities

## Sumário

- 01 - path-traversal
  - [01 - File path traversal, simple case](./01-path-traversal/01-simple-case/README.md)
- 02 - Access Control
  - [01 - Unprotected admin functionality](./02-access-control/01-unprotected-admin-functionality/README.md)
  - [02 - Unprotected admin functionality with unpredictable URL](./02-access-control/02-unprotected-admin-unpredictable-url/README.md)
  - [03 - User role controlled by request parameter](./02-access-control/03-user-role-controlled-by-request-parameter/README.md)
  - [04 - User ID controlled by request parameter, with unpredictable user IDs](./02-access-control/04-user-id-controlled-unpredictable-user-ids/README.md)
  - [05 - User ID controlled by request parameter with password disclosure](./02-access-control/05-user-id-controlled-with-password-disclosure/README.md)

- 03 - Authentication
  - [01 -Username enumeration via different responses](./03-authentication/01-username-enumeration-via-different-responses/README.md)
  - [02 - 2FA simple bypass](./03-authentication/02-2fa-simple-bypass/README.md)

- 04 - Server-side request forgery (SSRF)
  - [01 - Basic SSRF against the local server](./04-ssrf/01-basic-ssrf-against-localhost/README.md)
  - [02 - Basic SSRF against another back-end system](./04-ssrf/02-basic-ssrf-against-another-backend-system/README.md)

- 05 - File upload vulnerabilities
  - [01 - Remote code execution via web shell upload](./05-file-upload-vulnerabilities/01-remote-code-execution-via-web-shell-upload/README.md)
  - [02 - Web shell upload via Content-Type restriction bypass](./05-file-upload-vulnerabilities/02-web-shell-upload-via-content-type-bypass/README.md)

- 06 - OS command injection
  - [01 - OS command injection, simple case](./06-os-command-injection/01-simple-case/README.md)

- 07 - SQL injection
  - [01 - SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./07-sql-injection/01-retrieve-hidden-data/README.md)
  - [02 - SQL injection vulnerability allowing login bypass](./07-sql-injection/02-login-bypass/README.md)

---
[⬅ Voltar](../README.md)