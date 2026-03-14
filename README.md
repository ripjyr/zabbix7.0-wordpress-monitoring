# Zabbix Template: WordPress Security & Exposure Check

## Overview

This repository provides a monitoring template for detecting common security exposures and misconfigurations in **WordPress** websites using **Zabbix HTTP Agent checks**.

The template performs lightweight external checks against publicly accessible URLs to identify information leaks, exposed files, and potentially risky WordPress features.

It helps administrators quickly detect security misconfigurations and improve the security posture of their WordPress installations.

The template is designed for **low-impact monitoring** and does not require installing any agent or plugin on the monitored WordPress server.

Compatible with **Zabbix 7.0+**.

---

## Features

The template monitors the following potential exposure points:

| Check                         | Description                                         |
| ----------------------------- | --------------------------------------------------- |
| WordPress Admin Panel         | Detects if `/wp-admin/` is publicly reachable       |
| WordPress Login Page          | Detects if `/wp-login.php` is accessible            |
| Admin User Enumeration        | Checks `?author=1` redirect behavior                |
| Config Backup Exposure        | Detects `/wp-config.php.bak`                        |
| Debug Log Exposure            | Detects `/wp-content/debug.log`                     |
| Git Repository Exposure       | Detects `.git/` directory exposure                  |
| Install Script Exposure       | Detects `/wp-admin/install.php`                     |
| License File                  | Detects `license.txt`                               |
| Readme File                   | Detects `readme.html` version disclosure            |
| REST API Availability         | Detects `/wp-json/`                                 |
| User Enumeration via REST API | Detects `/wp-json/wp/v2/users`                      |
| XML-RPC Interface             | Detects `/xmlrpc.php`                               |
| WordPress Cron                | Detects `/wp-cron.php` accessibility                |
| WordPress Version             | Extracts version from RSS feed                      |
| Directory Listing Exposure    | Detects directory listing in `/wp-content/uploads/` |
| Setup Config Script Exposure  | Detects `/wp-admin/setup-config.php` availability   |
| PHP Information Page          | Detects exposed `phpinfo.php` or `info.php` pages   |
| Backup Directory Listing      | Detects directory listing in `/backup/` directory   |
| Old Directory Listing         | Detects directory listing in `/old/` directory      |
| Environment File Exposure    | Detects exposed `.env` configuration file      　　　　　　　　　　　　|

---

## Intended Use

This template is intended for:

* Security auditing
* External monitoring
* Security investigation

It helps identify publicly exposed resources and potential information leaks in WordPress installations.

---

## Scope

This template performs **passive external checks only**.

It does **not attempt to exploit vulnerabilities** or perform intrusive scanning.

All checks are limited to simple HTTP requests to publicly accessible endpoints.

---

## Requirements

* Zabbix 7.0 or later
* HTTP Agent enabled
* Internet access from the Zabbix server to the monitored website

---

## Installation

1. Download the template YAML file.

2. In the Zabbix Web UI navigate to:

```
Data collection → Templates → Import
```

3. Upload the YAML template.

4. Assign the template to the target host.

---

## Host Configuration

The template uses the following macro:

{$WP_PROTO} and {$WP_PORT} allow the template to support custom WordPress deployments using non-standard ports or HTTP/HTTPS configurations.

| Macro | Description | Default value |
|------|-------------|--------|
| {$WP_PROTO} | Protocol used for monitoring | https |
| {$WP_PORT} | Port used for monitoring | 443 |
| {$WPROOT} | WordPress installation path | / |

### Example configuration:

### {$WP_PROTO}
```
https
```

or
```
http
```
This macro defines the ** protocol ** of the WordPress HTTP server.

### Example configuration:

| Site URL                | Macro Value |
| ----------------------- | ----------- |
| https://example.com     | https       |
| http://example.com/     | http        |

### {$WP_PORT}
```
443
```

or

```
8443
```
This macro defines the **port number** of the WordPress HTTP server.

### Example configuration:

| Site URL                      | Macro Value |
| ----------------------------- | ------------ |
| https://example.com           | 443(defalut) |
| https://example.com:**8443**/     | 8443         |

### {$WPROOT}
```
/
```

or

```
/wp/
```

This macro defines the ** root path** of the WordPress installation.

### Example configuration:

| Site URL                | Macro Value |
| ----------------------- | ----------- |
| https://example.com/     | /           |
| https://example.com/wp/ | /wp/        |

---

## Trigger Severity Policy

The template uses the following severity classification:

Information  
Operational information such as WordPress version changes.

Warning  
Default WordPress behaviors that increase attack surface but
are commonly enabled.

Average  
Conditions that allow reconnaissance activities such as
user enumeration.

High  
Information exposure or server misconfiguration that may
leak sensitive information.

Disaster  
Critical conditions that may allow site takeover or indicate
an incomplete WordPress installation.

---

## Trigger Behavior

Triggers are generated when sensitive files or interfaces are publicly accessible.

Example alerts:

* WordPress config backup exposed
* WordPress debug log exposed
* WordPress git path exposed
* WordPress user enumeration possible
* WordPress version leak via readme.html
* WordPress directory listing enabled
* phpinfo page exposure
* backup directory listing
* old directory listing
* WordPress setup script exposure
* environment configuration file exposure (.env)
Most triggers require **manual close** to ensure administrators acknowledge the issue.

---

## Monitoring Method

All checks are performed using **HTTP HEAD or GET requests** via the Zabbix HTTP Agent.

HTTP status codes are extracted using regex preprocessing:

```
HTTP/[0-9\.]+\s([0-9]{3})
```

Directory listing detection is performed by scanning the response body for typical directory index markers:

```
Index of
Parent Directory
```

---

## Additional Security Checks

### WordPress Setup Configuration Script

The template checks whether the WordPress configuration setup script is publicly accessible.

/wp-admin/setup-config.php


This script is normally used only during the initial installation of WordPress.

If it becomes accessible after a site migration or configuration loss, the site may enter a setup state where WordPress could potentially be reconfigured.

This condition may indicate:

- Missing `wp-config.php`
- Incomplete WordPress migration
- Database connection failure

Administrators should verify that the WordPress installation is fully configured and that setup scripts are not accessible.

---

### PHP Information Page Exposure

The template checks for exposed PHP information pages such as:

```
/phpinfo.php
/info.php
```

These pages reveal detailed information about the server environment, including:

- PHP version
- loaded extensions
- filesystem paths
- server configuration

This information may assist attackers in identifying vulnerabilities.

If such pages are found, they should be removed from the web root after debugging.

---

### Backup and Old Directory Exposure

The template checks for directory listing in common backup locations:

```
/backup/
/old/
```

These directories are often created during site migrations or manual backup operations.

If directory listing is enabled, attackers may be able to download:

- backup archives
- database dumps
- full copies of previous website versions
- WordPress configuration files

These directories should not normally be accessible from the internet.

Administrators should remove them from the web root or disable directory listing in the web server configuration.

---

## Composite Security Triggers

In addition to individual exposure checks, this template also detects
**combined attack surface conditions**.

These composite triggers identify situations where multiple WordPress
features are exposed simultaneously, significantly increasing the
risk of brute-force attacks.

### XML-RPC + User Enumeration

If both of the following conditions are detected:

- XML-RPC interface accessible (`/xmlrpc.php`)
- User enumeration via REST API (`/wp-json/wp/v2/users`)

Attackers may be able to identify valid usernames and perform
high-speed authentication attempts using XML-RPC multicall.

### XML-RPC + User Enumeration + Public Login Page

If the following conditions are detected simultaneously:

- XML-RPC enabled
- REST API user enumeration possible
- WordPress login page accessible (`/wp-login.php`)

This combination significantly increases the attack surface for
automated brute-force attacks against WordPress accounts.

These conditions are treated as **high severity security risks**
by this template.

---

## Tested Environment

* WordPress 6.x
* Zabbix 7.0
* Nginx / Apache web servers

---

## License

This project is released under the **MIT License**.

---

## Disclaimer

This template is provided for monitoring and security awareness purposes.

Use it at your own risk.

---

## Author

Community contribution for WordPress exposure monitoring using Zabbix.

---

## Project Repository

Source code and updates are available on GitHub:

https://github.com/ripjyr/zabbix7.0-wordpress-monitoring
