# HTTP Enumeration

## Service Information

- **Service:** HTTP
- **Port:** TCP/80
- **Software:** Apache HTTP Server
- **Version:** Apache 2.2.8
- **Target:** Metasploitable 2

## 1. HTTP Service Discovery

The HTTP service was identified using Nmap:

```bash
nmap 192.168.56.20 -p 80 -sV
```

Nmap identified TCP/80 as an open HTTP service running Apache:

```text
80/tcp open http Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```

![View Nmap HTTP service detection evidence](../evidence/06_HTTP01_nmap_http_detection.png)

The detected version identifies the web server as an obsolete Apache HTTP Server implementation.

## 2. HTTP Header Inspection

The HTTP service was then queried directly using `curl`:

```bash
curl -I http://192.168.56.20/
```

The `-I` option requests only the HTTP response headers without retrieving the complete page.

The response disclosed information including:

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html
```

![View HTTP header evidence](../evidence/06_HTTP02_http_headers.png)

The response therefore disclosed both the Apache web-server implementation/version and the PHP version.

This information is useful during reconnaissance because it identifies components that may require further security review.

## 3. Web Content and Information Disclosure

The web application's HTML content was retrieved using:

```bash
curl http://192.168.56.20/
```

The returned page disclosed information about the Metasploitable installation and linked directly to several hosted applications:

```text
/twiki/
/phpMyAdmin/
/mutillidae/
/dvwa/
/dav/
```

![View web content and information disclosure evidence](../evidence/06_HTTP03_banner_warning_sensitiveinfo_numeration%20%28REDACTED%29.png)

The presence of application links directly in the HTTP response demonstrates that the web server exposes multiple applications through the same HTTP service.

The page also contained information about the laboratory environment and default credentials. Credentials have been redacted in the public evidence screenshot.

The disclosed information is relevant from a security-assessment perspective because it provides an unauthenticated user with additional information about the target environment and the applications exposed through the web server.

## 4. Web Directory Enumeration

Gobuster was used to enumerate additional resources exposed by the HTTP service:

```bash
gobuster dir -u http://192.168.56.20/ -w /usr/share/wordlists/dirb/common.txt
```

The enumeration identified multiple resources.

Accessible resources returning HTTP 200 included:

```text
/index
/index.php
/phpinfo.php
/phpinfo
/test
```

Additional resources returning HTTP 301 redirects included:

```text
/dav/
/phpMyAdmin/
/twiki/
```

The scan also identified restricted resources returning HTTP 403:

```text
/.htaccess
/.htpasswd
/.hta
/cgi-bin/
/server-status
```

![View Gobuster directory enumeration evidence](../evidence/06_HTTP04_gobuster_directory_enumeration.png)

The HTTP 403 responses are important because they demonstrate that these resources exist while access to them is currently denied.

The assessment does not claim that the contents of these restricted resources were accessible.

### phpinfo Exposure

The enumeration identified:

```text
/phpinfo.php
/phpinfo
```

PHP information pages can disclose extensive technical information about the server environment, including PHP configuration, loaded modules, paths and other implementation details.

The presence of an accessible PHP information page should therefore be reviewed as a potential information-disclosure issue.

## 5. phpMyAdmin Identification

The discovered `/phpMyAdmin/` resource was inspected to confirm the application hosted at the endpoint.

The application was confirmed to be phpMyAdmin through its HTTP response and HTML content.

![View phpMyAdmin identification evidence](../evidence/06_HTTP05_phpMyAdmin_identification.png)

The assessment at this stage was limited to application identification.

No authentication bypass, exploitation or unauthorized database access was attempted.

## 6. Browser-Based Web Access

The HTTP service was also accessed through a web browser:

```text
http://192.168.56.20/
```

The browser displayed the Metasploitable web interface and its exposed application links.

![View browser-based HTTP access evidence](../evidence/06_HTTP06_web_access.png)

This provides visual confirmation that the web service and its linked applications were accessible through a standard HTTP client.

## Enumeration Summary

The HTTP assessment established the following characteristics:

- TCP/80 was exposed on the target.
- Apache HTTP Server 2.2.8 was identified.
- The server disclosed Apache and PHP version information through HTTP headers.
- The root web page disclosed multiple hosted applications.
- Gobuster identified multiple accessible and restricted web resources.
- `/phpinfo.php` and `/phpinfo` were identified as potentially sensitive information-disclosure endpoints.
- `/phpMyAdmin/` was confirmed to be accessible.
- `/twiki/`, `/mutillidae/`, `/dvwa/` and `/dav/` were identified as additional hosted resources.
- Several resources returned HTTP 403, demonstrating that they existed but were access-restricted.
- The web service was successfully accessed through both command-line tools and a standard browser.
- Credentials visible in the original laboratory page were redacted in the public evidence.

## Security Considerations

The enumeration identified several areas requiring security review:

- Legacy Apache and PHP versions.
- Information disclosure through HTTP headers.
- Information disclosure through the root web page.
- Multiple applications exposed through a single web server.
- Accessible PHP information pages.
- Administrative or application interfaces such as phpMyAdmin.
- Additional resources identified through directory enumeration.

The presence of a resource does not by itself demonstrate a vulnerability. Each application and endpoint should be reviewed according to its intended purpose, authentication requirements and security configuration.

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

The HTTP assessment focused on service identification, HTTP response analysis, web-content inspection, directory enumeration and application identification.

No exploitation, authentication bypass, unauthorized database access or persistence was performed during this assessment.

## Evidence Summary

The HTTP evidence consists of six screenshots:

```text
06_HTTP01_nmap_http_detection.png
06_HTTP02_http_headers.png
06_HTTP03_banner_warning_sensitiveinfo_numeration (REDACTED).png
06_HTTP04_gobuster_directory_enumeration.png
06_HTTP05_phpMyAdmin_identification.png
06_HTTP06_web_access.png
```

The evidence chain is:

**Service identification → HTTP header analysis → web-content inspection → directory enumeration → application identification → browser verification**
