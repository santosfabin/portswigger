# Web Security Academy — Caderno de Estudos e Laboratórios

Repositório dedicado à documentação técnica, reprodução e análise de causa raiz dos laboratórios da [PortSwigger Web Security Academy](https://portswigger.net/web-security).

O objetivo deste material não é apenas registrar payloads de exploração, mas dissecar a mecânica da falha no tráfego HTTP, o comportamento do backend/frontend e as formas de mitigação no código-fonte.

## Server-side vulnerabilities

### 01 - path-traversal

- [01 - File path traversal, simple case](./server-side-vulnerabilities/01-path-traversal/01-simple-case/README.md)

### 02 - Access Control

- [01 - Unprotected admin functionality](./server-side-vulnerabilities/02-access-control/01-unprotected-admin-functionality/README.md)
- [02 - Unprotected admin functionality with unpredictable URL](./server-side-vulnerabilities/02-access-control/02-unprotected-admin-unpredictable-url/README.md)
- [03 - User role controlled by request parameter](./server-side-vulnerabilities/02-access-control/03-user-role-controlled-by-request-parameter/README.md)
- [04 - User ID controlled by request parameter, with unpredictable user IDs](./server-side-vulnerabilities/02-access-control/04-user-id-controlled-unpredictable-user-ids/README.md)
- [05 - User ID controlled by request parameter with password disclosure](./server-side-vulnerabilities/02-access-control/05-user-id-controlled-with-password-disclosure/README.md)

### 03 - Authentication

- [01 -Username enumeration via different responses](./server-side-vulnerabilities/03-authentication/01-username-enumeration-via-different-responses/README.md)
- [02 - 2FA simple bypass](./server-side-vulnerabilities/03-authentication/02-2fa-simple-bypass/README.md)

### 04 - Server-side request forgery (SSRF)

- [01 - Basic SSRF against the local server](./server-side-vulnerabilities/04-ssrf/01-basic-ssrf-against-localhost/README.md)
- [02 - Basic SSRF against another back-end system](./server-side-vulnerabilities/04-ssrf/02-basic-ssrf-against-another-backend-system/README.md)

### 05 - File upload vulnerabilities

- [01 - Remote code execution via web shell upload](./server-side-vulnerabilities/05-file-upload-vulnerabilities/01-remote-code-execution-via-web-shell-upload/README.md)
- [02 - Web shell upload via Content-Type restriction bypass](./server-side-vulnerabilities/05-file-upload-vulnerabilities/02-web-shell-upload-via-content-type-bypass/README.md)

### 06 - OS command injection

- [01 - OS command injection, simple case](./server-side-vulnerabilities/06-os-command-injection/01-simple-case/README.md)

### 07 - SQL injection

- [01 - SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./server-side-vulnerabilities/07-sql-injection/01-retrieve-hidden-data/README.md)
- [02 - SQL injection vulnerability allowing login bypass](./server-side-vulnerabilities/07-sql-injection/02-login-bypass/README.md)

---

## SQL injection

### 01 - Retrieving hidden data

- [01 - SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./sql-injection/01-retrieving-hidden-data/01sql-injection-vulnerability-in-where-clause-allowing-retrieval-of-hidden-data/README.md)

### 02 - Subverting application logic

- [01 - SQL injection vulnerability allowing login bypass](./sql-injection/02-subverting-application-logic/01-sql-injection-vulnerability-allowing-login-bypass/README.md)

### 03 - Determining the number of columns required

- [01 - SQL injection UNION attack, determining the number of columns returned by the query](./sql-injection/03-determining-the-number-of-columns-required/01-sql-injection-union-attack-determining-the-number-of-columns-returned-by-the-query/README.md)

### 04 - Finding columns with a useful data type

- [01 - SQL injection UNION attack, finding a column containing text](./sql-injection/04-finding-columns-with-a-useful-data-type/01-sql-injection-union-attack-finding-a-column-containing-text/README.md)

### 05 - Using a SQL injection UNION attack to retrieve interesting data

- [01 - SQL injection UNION attack, retrieving data from other tables](./sql-injection/05-using-a-sql-injection-union-attack-to-retrieve-interesting/01-sql-injection-union-attack-retrieving-data-from-other-tables/README.md)

### 06 - Retrieving multiple values within a single column

- [01 - SQL injection UNION attack, retrieving multiple values in a single column](./sql-injection/06-retrieving-multiple-values-within-a-single-column/01-sql-injection-union-attack-retrieving-multiple-values-in-a-single-column/README.md)

### 07 - Examining the database

- [01 - SQL injection attack, querying the database type and version on MySQL and Microsoft](./sql-injection/07-examining-the-database/01-sql-injection-attack-querying-the-database-type-and-version-on-mysql-and-microsoft/README.md)
- [02 - SQL injection attack, listing the database contents on non-Oracle databases](./sql-injection/07-examining-the-database/02-sql-injection-attack-listing-the-database-contents-on-non-oracle-databases/README.md)

### 08 - Exploiting blind SQL injection by triggering conditional responses

- [01 - Blind SQL injection with conditional responses](./sql-injection/08-blind-sql-injection/01-blind-sql-injection-with-conditional-responses/README.md)

### 09 - Error-based SQL injection

- [01 - Blind SQL injection with conditional errors](./sql-injection/09-error-based-sql-injection/01-blind-sql-injection-with-conditional-errors/README.md)
- [02 - Visible error-based SQL injection](./sql-injection/09-error-based-sql-injection/02-visible-error-based-sql-injection/README.md)

### 10 - Exploiting blind SQL injection by triggering time delays

- [01 - Blind SQL injection with time delays and information retrieval](./sql-injection/10-exploiting-blind-sql-injection-by-triggering-time-delays/01-blind-sql-injection-with-time-delays-and-information-retrieval/README.md)

### 11 - SQL injection in different contexts

- [01 - SQL injection with filter bypass via XML encoding](./sql-injection/11-sql-injection-in-different-contexts/01-sql-injection-with-filter-bypass-via-xml-encoding/README.md)
