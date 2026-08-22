# Tomcat Enumeration

## Service Information

- **Service:** Apache Tomcat
- **Port:** TCP/8180
- **Version identified:** Apache Tomcat 5.5
- **HTTP server:** Apache-Coyote/1.1
- **Management interface:** Tomcat Manager Application
- **Authentication:** HTTP Basic Authentication
- **Credentials validated:** `tomcat:tomcat`
- **Target:** Metasploitable 2
- **Assessment approach:** Service identification, web-interface enumeration, authentication validation and management-interface review. No exploitation or code execution was performed.

---

## 1. Tomcat Service Discovery

We first identified the service running on TCP/8180.

### Command used

```bash
nmap -p 8180 -sV 192.168.56.20
```

Nmap identified:

```text
8180/tcp open http Apache Tomcat/Coyote JSP engine 1.1
```

This confirmed that the target was exposing an HTTP service associated with Apache Tomcat on TCP/8180.

![Nmap Tomcat service detection](../evidence/11_TOMCAT01_nmap_tomcat_detection.png)

The Nmap result identified the HTTP server as:

```text
Apache Tomcat/Coyote JSP engine 1.1
```

Further enumeration of the web interface was therefore performed.

---

## 2. Accessing the Tomcat Web Interface

We then queried the Tomcat HTTP service directly.

### Command used

```bash
curl http://192.168.56.20:8180/
```

The response revealed the Tomcat default web page and identified the server as:

```text
Apache Tomcat/5.5
```

The response also exposed links to several Tomcat resources, including:

```text
manager/status
admin
manager/html
```

This established that the target was exposing not only the Tomcat web service itself, but also management-related applications.

![Tomcat default web page](../evidence/11_TOMCAT02_tomcat_default_page.png)

The presence of `/manager/html` was particularly relevant because it indicated that the Tomcat Manager application was available for further assessment.

---

## 3. Tomcat Manager Authentication Discovery

We then checked whether the Tomcat Manager application was accessible.

### Command used

```bash
curl -I http://192.168.56.20:8180/manager/html
```

The server returned:

```text
HTTP/1.1 401 Unauthorized
```

and:

```text
WWW-Authenticate: Basic realm="Tomcat Manager Application"
```

This confirmed that:

- The Tomcat Manager application existed.
- The application was remotely reachable.
- Authentication was required.
- The application used HTTP Basic Authentication.

![Tomcat Manager authentication](../evidence/11_TOMCAT03_tomcat_manager_authentication.png)

The authentication mechanism was therefore identified before attempting authenticated access.

---

## 4. Authenticated Access to Tomcat Manager

We then tested the Tomcat credentials available in the laboratory environment.

### Command used

```bash
curl -u tomcat:tomcat http://192.168.56.20:8180/manager/html
```

The request successfully returned the Tomcat Manager HTML interface.

This confirmed that:

```text
tomcat:tomcat
```

was accepted by the Tomcat Manager application and provided authenticated access to the management interface.

![Authenticated Tomcat Manager session](../evidence/11_TOMCAT04_tomcat_manager_authenticated.png)

The assessment therefore established that the Manager interface was not only exposed but also accessible using the tested credentials.

No attempt was made to obtain credentials through exploitation.

---

## 5. Retrieving the Manager Interface with wget

The authenticated Manager interface was also retrieved using `wget`.

### Command used

```bash
wget \
--user=tomcat \
--password=tomcat \
http://192.168.56.20:8180/manager/html
```

The initial request returned:

```text
401 Unauthorized
```

The client then authenticated successfully and received:

```text
200 OK
```

The response was saved locally as:

```text
html
```

This demonstrated that the authenticated Tomcat Manager interface could also be retrieved directly from the command line.

![Tomcat Manager retrieved with wget](../evidence/11_TOMCAT05_tomcat_manager_wget.png)

The downloaded HTML was subsequently inspected to identify the functionality exposed by the authenticated management interface.

---

## 6. Enumerating Tomcat Manager Deployment Functionality

After retrieving the Manager interface, we inspected the HTML locally.

### Search for deployment functionality

```bash
grep -i deploy html
```

The output revealed:

```text
Deploy directory or WAR file located on server
```

and:

```text
WAR file to deploy
```

### Search for WAR functionality

```bash
grep -i war html
```

The output revealed:

```text
WAR or Directory URL:
```

and:

```text
Select WAR file to upload
```

### Search for upload functionality

```bash
grep -i upload html
```

The output revealed an upload form targeting:

```text
/manager/html/upload
```

including:

```text
<form action="/manager/html/upload"
method="post" enctype="multipart/form-data">
```

![Tomcat Manager deployment functionality](../evidence/11_TOMCAT06_tomcat_manager_deployment_functionality.png)

This confirmed that the authenticated Tomcat Manager interface exposed administrative functionality for deploying applications and uploading WAR files.

At this stage, the assessment had established the functionality available to the authenticated account without needing to exploit it.

---

## 7. Controlled Upload-Endpoint Test

A controlled test was subsequently performed against the identified upload endpoint.

A harmless text file was created locally:

```bash
echo "hello" > test2.txt
```

The file was then submitted to the Tomcat Manager upload endpoint:

```bash
curl \
-u tomcat:tomcat \
-F "deployWar=@test2.txt" \
http://192.168.56.20:8180/manager/html/upload
```

The server returned the Tomcat Manager HTML interface.

![Tomcat controlled upload-endpoint test](../evidence/11_TOMCAT07_tomcat_tentative_malicious_deployment.png)

This test was deliberately limited to a harmless text file.

It did **not** involve deployment of a malicious WAR file, code execution or persistence.

The purpose of the test was to document the exposed management functionality and validate interaction with the upload endpoint without progressing into exploitation.

---

# Tomcat Enumeration Summary

The assessment established that:

- Apache Tomcat was exposed on TCP/8180.
- Nmap identified the service as `Apache Tomcat/Coyote JSP engine 1.1`.
- The web interface identified the server as Apache Tomcat 5.5.
- The Tomcat Manager application was remotely accessible.
- The Manager application used HTTP Basic Authentication.
- Authentication was required to access `/manager/html`.
- The credentials `tomcat:tomcat` successfully authenticated.
- The authenticated Manager interface exposed deployment functionality.
- WAR deployment functionality was identified.
- A WAR upload form was identified at `/manager/html/upload`.
- The upload endpoint was tested using a harmless text file.
- No malicious WAR was deployed.
- No code execution or persistence was attempted.

The evidence chain is:

**Service identification → Tomcat version identification → Manager discovery → authentication mechanism identification → authenticated Manager access → deployment functionality enumeration → controlled upload-endpoint test**

---

## Security-Relevant Observations

The enumeration identified several areas requiring further security assessment:

### Legacy Tomcat Version

The web interface identified:

```text
Apache Tomcat/5.5
```

This should be investigated as a legacy software component.

### Exposed Management Interface

The Tomcat Manager application was accessible remotely through:

```text
http://192.168.56.20:8180/manager/html
```

The exposure and access controls of the management interface should therefore be reviewed.

### Weak Credentials

The credentials:

```text
tomcat:tomcat
```

successfully authenticated to the Manager application.

This should be assessed as an authentication and access-control issue in the findings section.

### Administrative Deployment Functionality

The authenticated Manager interface exposed:

```text
Deploy
```

and:

```text
WAR file to deploy
```

as well as the upload endpoint:

```text
/manager/html/upload
```

This should be assessed separately because application deployment represents administrative functionality.

---

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

Testing was limited to:

- Service identification.
- Version identification.
- Web-interface enumeration.
- Tomcat Manager discovery.
- Authentication mechanism identification.
- Authentication validation.
- Management-interface enumeration.
- Deployment functionality identification.
- Controlled interaction with the upload endpoint.

No malicious WAR file was deployed.

No code execution was attempted.

No persistence was established.

No destructive modification was performed.

The controlled upload test used a harmless text file solely to validate interaction with the exposed management functionality.

---

## Evidence Summary

The Tomcat assessment uses seven screenshots:

```text
11_TOMCAT01_nmap_tomcat_detection.png
11_TOMCAT02_tomcat_default_page.png
11_TOMCAT03_tomcat_manager_authentication.png
11_TOMCAT04_tomcat_manager_authenticated.png
11_TOMCAT05_tomcat_manager_wget.png
11_TOMCAT06_tomcat_manager_deployment_functionality.png
11_TOMCAT07_tomcat_tentative_malicious_deployment.png
```

The evidence represents the following stages:

1. **Nmap** — Tomcat service identification.
2. **curl** — Tomcat default web interface and version identification.
3. **curl -I** — Manager authentication requirement.
4. **curl with credentials** — authenticated Manager access.
5. **wget** — authenticated Manager retrieval from the command line.
6. **grep** — deployment and WAR upload functionality identified.
7. **curl upload test** — controlled interaction with the upload endpoint.

The complete evidence chain is:

**Tomcat discovery → version identification → Manager discovery → authentication validation → functionality enumeration → controlled upload test**

---

## Commands

### Tomcat service detection

```bash
nmap -p 8180 -sV 192.168.56.20
```

### Retrieve the default Tomcat page

```bash
curl http://192.168.56.20:8180/
```

### Check Manager authentication

```bash
curl -I http://192.168.56.20:8180/manager/html
```

### Authenticate to Tomcat Manager

```bash
curl -u tomcat:tomcat \
http://192.168.56.20:8180/manager/html
```

### Download Manager page with wget

```bash
wget \
--user=tomcat \
--password=tomcat \
http://192.168.56.20:8180/manager/html
```

### Search Manager HTML for deployment functionality

```bash
grep -i deploy html
```

### Search for WAR deployment functionality

```bash
grep -i war html
```

### Search for upload functionality

```bash
grep -i upload html
```

### Create harmless test file

```bash
echo "hello" > test2.txt
```

### Test the upload endpoint

```bash
curl \
-u tomcat:tomcat \
-F "deployWar=@test2.txt" \
http://192.168.56.20:8180/manager/html/upload
```

---

