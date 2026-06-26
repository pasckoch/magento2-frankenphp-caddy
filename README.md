# Magento 2 Caddyfile Configuration for FrankenPHP / Caddy

An enterprise-grade, fully optimized, and secure `Caddyfile` configuration for running **Magento 2** seamlessly on **FrankenPHP** or native **Caddy v2**. 

This configuration is a faithful translation of Magento's official `nginx.conf.sample`. It completely resolves notorious production issues like cascading static asset signatures (versioned assets), correct MIME types (`requirejs-config.js`), and strict directory security rules.

---

### 📖 Technical Documentation & Architecture Report
For an in-depth analysis of how this setup bridges the gap between Magento and FrankenPHP, read the full official report here:  
👉 **[https://pascalkoch.net/home/index/5/en](https://pascalkoch.net/home/index/5/en)**

---

## 🚀 Features

* **Production-Ready Static Fallbacks**: Emulates Nginx's strict `try_files` and `rewrite ... last` cascading logic for static files.
* **Asset Signature Stripping**: Automated stripping of `/static/versionXXXX/` tokens without losing physical file references.
* **Bulletproof Security**: Out-of-the-box blocklists for sensitive files (`.git`, `.htaccess`, `.user.ini`, system XMLs, customer/downloadable media folders).
* **FrankenPHP Optimized**: Seamlessly injects Magento's required environment variables (`memory_limit`, `max_execution_time`, `session.auto_start`).
* **Built-in Performance**: Native HTTP/3 support (via Caddy), modern encoding (`zstd`, `gzip`), and aggressive client-side caching headers for static/media streams.
* **Custom Error Fallbacks**: Automatic routing of `403` and `404` server errors to Magento's native error handling engine (`/errors/404.php`).

## 📋 Permissions Best Practice

Before deploying, ensure your project permissions are correct. To prevent CLI vs. Web Server user collision issues on Linux, it is highly recommended to enforce **ACLs (Access Control Lists)** on runtime directories:

```bash
# Set up bi-directional permissions for your CLI user and the Web server user (e.g., frankenphp or caddy)
sudo setfacl -R -m u:YOUR_CLI_USER:rwx -m u:frankenphp:rwx var/ generated/ pub/static/ pub/media/
sudo setfacl -R -d -m u:YOUR_CLI_USER:rwx -m u:frankenphp:rwx var/ generated/ pub/static/ pub/media/

⚙️ Installation

    Copy the Caddyfile from this repository.

    Place it in your FrankenPHP/Caddy configuration drop-in folder (e.g., /etc/frankenphp/Caddyfile.d/magento2.caddyfile) or merge it into your global master Caddyfile.

    Update the generic variables to match your setup:

        Change magento2.localhost.com to your target local or production domain.

        Update root * /var/www/html/magento2/pub to point to your absolute path to Magento's pub/ folder.

    Reload your application server:

```bash

sudo systemctl reload frankenphp
# or caddy reload --config /etc/caddy/Caddyfile

🛠️ Configuration Architecture Breakdown

Unlike basic setups that break JavaScript dependencies by passing everything straight to PHP, this file acts as a structured linear middleware stack:

    Global Security Filters: Intercepts illegal access early to save PHP processing cycles. It blocks hidden dotfiles, VCS directories (.git), and critical runtime configurations (.user.ini, .htaccess).

    Protected Deployment Workflows: Isolates /setup and /update flows, allowing web-based upgrades only under controlled network conditions or explicit exclusions.

    Static Engine Route (/static/*):

        Phase 1 (Pre-routing): Strips asset signatures (e.g., /static/version12345/...) globally at the front edge of the request to normalize internal pointers.

        Phase 2 (File System Evaluation): Checks the local filesystem for the clean path. If found, it drops out of the rewrite tree and serves it with aggressive headers (Cache-Control: public, max-age=31536000, immutable).

        Phase 3 (Fallback Router): If missing (such as dynamic requirejs-config.js files or untracked resources), it streams the request directly into /static.php to handle on-the-fly compiling.

    Media Engine Route (/media/*): Enforces safety constraints on dynamic asset lookups, blocks execution of arbitrary scripts or custom theme layout XML configurations, and handles fallbacks to get.php for dynamic catalog image cropping and generation.

    Global Fallback: Standard catch-all routing implementing native clean URLs by passing unmapped application routes to the primary index.php entrypoint alongside appropriate memory limit overrides.

🤝 Contributing

Contributions, issues, and feature requests are welcome! If you find edge cases on specific Magento Open Source or Adobe Commerce versions, feel free to open an issue or submit a Pull Request.
📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
