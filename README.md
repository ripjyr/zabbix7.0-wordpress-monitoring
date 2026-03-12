# Zabbix Template: WordPress Security & Exposure Check

## Overview

This repository provides a monitoring template for detecting common security exposures and misconfigurations in **WordPress** websites using **Zabbix HTTP Agent checks**.

The template performs external checks against publicly accessible URLs to identify information leaks, exposed files, and potentially risky WordPress features.

It is designed to help administrators quickly detect misconfigurations and improve the security posture of their WordPress installations.

This template is compatible with **Zabbix 7.0+**.

---

## Features

The template monitors the following potential exposure points:

| Check                         | Description                                   |
| ----------------------------- | --------------------------------------------- |
| WordPress Admin Panel         | Detects if `/wp-admin/` is publicly reachable |
| WordPress Login Page          | Detects if `/wp-login.php` is accessible      |
| Admin User Enumeration        | Checks `?author=1` redirect behavior          |
| Config Backup Exposure        | Detects `/wp-config.php.bak`                  |
| Debug Log Exposure            | Detects `/wp-content/debug.log`               |
| Git Repository Exposure       | Detects `.git/` directory exposure            |
| Install Script Exposure       | Detects `/wp-admin/install.php`               |
| License File                  | Detects `license.txt`                         |
| Readme File                   | Detects `readme.html` version disclosure      |
| REST API Availability         | Detects `/wp-json/`                           |
| User Enumeration via REST API | Detects `/wp-json/wp/v2/users`                |
| XML-RPC Interface             | Detects `/xmlrpc.php`                         |
| WordPress Cron                | Detects `/wp-cron.php` accessibility          |
| WordPress Version             | Extracts version from RSS feed                |

---

## Requirements

* Zabbix 7.0 or later
* HTTP Agent enabled
* Internet access from the Zabbix server to the monitored website

---

## Installation

1. Download the template YAML file.

2. In Zabbix Web UI:

```
Data collection → Templates → Import
```

3. Upload the YAML template file.

4. Assign the template to the target host.

---

## Host Configuration

The template uses the following macro:

```
{$WPROOT}
```

Example values:

```
/
```

or

```
/wordpress/
```

This macro defines the root path of the WordPress installation.

Example:

| Site URL                | Macro Value |
| ----------------------- | ----------- |
| https://example.com     | /           |
| https://example.com/wp/ | /wp/        |

---

## Trigger Behavior

Triggers are generated when sensitive files or interfaces are publicly accessible.

Example alerts:

* WordPress config backup exposed
* WordPress debug log exposed
* WordPress git path exposed
* WordPress user enumeration possible
* WordPress version leak via readme.html

Most triggers require **manual close** to ensure administrators acknowledge the issue.

---

## Monitoring Method

All checks are performed using **HTTP HEAD or GET requests** via Zabbix HTTP Agent.

HTTP status codes are extracted using regex preprocessing.

Example:

```
HTTP/[0-9\.]+\s([0-9]{3})
```

---

## Security Note

This template detects **exposed resources**, but it does not exploit vulnerabilities.

Detected items should be reviewed and secured according to WordPress security best practices.

---

## Tested Environment

* WordPress 6.x
* Zabbix 7.0
* Nginx / Apache web servers

---

## License

This project is released under the MIT License.

---

## Disclaimer

This template is provided for monitoring and security awareness purposes.
Use it at your own risk.

---

## Author

Community contribution for WordPress exposure monitoring using Zabbix.
