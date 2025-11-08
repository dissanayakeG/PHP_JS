# PHP Advance

# Security tips
- to prevent derectory listning, use public and private direcotories,
- use empty index.php in protected directories OR
- use .htaccess file and `Options -Indexes` entry in the file
- try to use .php extension for all the files, otherwise file can be accessed and see the data
> ex) `http.domain.com/private/config.json`

# Create a PHP Server

```bash
php -S localhost:<port> -> php -S localhost:8000 # -S should be uppercase
php -S localhost:8000 -t public
#-t public -> Sets the document root (the folder PHP serves) to the public/ directory.
```

# .htaccess (hypertext access)

- is a configuration file used by Apache web servers to control the behavior of your website without editing the main Apache configuration. It’s particularly useful for PHP developers because you can tweak site rules—security, redirects, caching, URL rewriting, etc.—on a per-directory basis.

## What `.htaccess` Does

- When placed in your site’s root or a subdirectory, `.htaccess` lets you
> Override default Apache settings
> Define custom error pages
> Rewrite URLs (e.g., for “pretty” URLs)
> Control access (authentication, IP restrictions)
> Redirect requests
> Set PHP configurations (if `AllowOverride` is enabled in Apache)
> Caching and Compression
> Password Protection

## Example `.htaccess` file
```
# ============================================================
# Enable URL rewriting
# ============================================================
# Turn on the rewrite engine so we can use RewriteRule below.
RewriteEngine On

# ============================================================
# Force HTTPS (redirect all HTTP requests to HTTPS)
# ============================================================
RewriteCond %{HTTPS} !=on
RewriteRule ^(.)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# ============================================================
# Redirect to "www" version (optional — choose one)
# ============================================================
# RewriteCond %{HTTP_HOST} !^www\. [NC]
# RewriteRule ^(.)$ https://www.%{HTTP_HOST}/$1 [L,R=301]

# ============================================================
# Remove "index.php" from URLs for clean routing
# ============================================================
# Example: example.com/index.php/about → example.com/about
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.)$ index.php?route=$1 [L,QSA]

# ============================================================
# Deny direct access to sensitive files
# ============================================================
<FilesMatch "\.(env|json|config|log|sh)$">
    Require all denied
</FilesMatch>

# ============================================================
# Set custom error pages
# ============================================================
ErrorDocument 404 /errors/404.html
ErrorDocument 500 /errors/500.html

# ============================================================
# Enable browser caching for static files
# ============================================================
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType text/css "access plus 1 week"
    ExpiresByType application/javascript "access plus 1 week"
</IfModule>

# ============================================================
# Enable GZIP compression for faster load times
# ============================================================
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/javascript
</IfModule>

# ============================================================
# Basic directory password protection (optional)
# ============================================================
# Protect sensitive admin areas by requiring login
# <Directory "/var/www/html/admin">
#     AuthType Basic
#     AuthName "Restricted Area"
#     AuthUserFile /path/to/.htpasswd
#     Require valid-user
# </Directory>

# ============================================================
# Override some PHP configurations (if allowed)
# ============================================================
php_value upload_max_filesize 20M
php_value post_max_size 25M
php_flag display_errors Off

```

## test and debug `.htaccess` rules properly

1. Make Sure `.htaccess` Is Even Working
- By default, Apache sometimes ignores `.htaccess` files. You need to ensure your site’s configuration allows them.

Check Apache Config
- Open your Apache site config (e.g., `/etc/apache2/sites-available/000-default.conf` or your project’s `.conf` file) and find your site’s `<Directory>` block. It should look like this:

```apache
<Directory /var/www/html>
    AllowOverride All
    Require all granted
</Directory>
 # `AllowOverride All` → lets `.htaccess` override settings (like rewrite rules).
 # Restart Apache after editing:
   sudo systemctl restart apache2
#If `AllowOverride` is `None`, `.htaccess` does nothing — period.
```

2. Test That It’s Active

Simple Test
```
#Create a temporary `.htaccess` in your web root with
Options -Indexes
#Then put a random file in that folder (like `test.txt`), and open that folder URL in a browser 
e.g.:http://localhost/test-folder/
#If you get a 403 Forbidden instead of a directory listing, `.htaccess` is working.
```

3. Debug Rewrite Rules

- If your rewrites aren’t behaving, turn on rewrite logging temporarily.
- Add this inside your `.htaccess`:

```apache
RewriteEngine On
RewriteLog "/var/log/apache2/rewrite.log"
RewriteLogLevel 3

#Then check the log

sudo tail -f /var/log/apache2/rewrite.log

#Note: On newer Apache versions (2.4+), `RewriteLog` is deprecated. Instead, use this in your main config (not `.htaccess`):

LogLevel alert rewrite:trace3

#Then view logs in:

sudo tail -f /var/log/apache2/error.log
```
- You’ll see detailed rewrite traces showing what rules triggered.

4. Use Built-In Testing Tools

- `curl` for Redirects
- Test your redirects and rewrites directly in the terminal:
```bash
curl -I http://example.com/old-page
```
- You should see something like:
```
HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-page
```

5. Check for Rule Conflicts

- If something fails:
- Comment out all rules and re-enable one at a time.
- Make sure there’s no nested `.htaccess` overriding your rules in subdirectories.
- Ensure modules are enabled:

  ```bash
  sudo a2enmod rewrite
  sudo a2enmod headers
  sudo a2enmod expires
  sudo a2enmod deflate
  sudo systemctl restart apache2
  ```
  
6. Validate Access Restrictions

- If you’re blocking `.env`, `.json`, or `.log` files:
```bash
curl -I http://example.com/.env
```
- Should return:
```
HTTP/1.1 403 Forbidden
```
- If it returns `200 OK`, your rule or `AllowOverride` setting is wrong.

7. Enable Error Reporting Temporarily

- During testing only — not production:

```apache
php_flag display_errors On
```
- Then trigger errors intentionally to verify your `.htaccess` is being read (like calling a missing PHP function). Once confirmed, turn it off.

8. Use a Local Debug Page

- Create a `rewrite-test.php` with:
```php
<?php
echo "<pre>";
print_r($_SERVER);
```
- This shows what Apache actually sends to PHP (like `REQUEST_URI` and `REDIRECT_URL`). It’s perfect for debugging rewrite variable mismatches.

9. Typical Apache Module Check

- To confirm which modules are active:
```bash
apache2ctl -M | grep rewrite
```
- If you don’t see `rewrite_module (shared)`, that’s your problem.

# File Importing in PHP

## include, include_once, require, require_once

 `include`  Loads and executes a file.
  If the file doesn’t exist → shows a warning but continues executing the rest of the script.

 `require`  Loads and executes a file.
  If the file doesn’t exist → throws a fatal error and stops execution.

 `include_once`  Works like `include`, but prevents re-including the same file.
  Useful to avoid “cannot redeclare function/variable/class” warnings.

 `require_once`  Works like `require`, but ensures the file is included only once.
  Commonly used in large projects to safely load config or class files.

> When your project grows, using `require_once` everywhere can make the code messy.
> Instead, you can use `spl_autoload_register()` to automatically load classes on demand.

# PHP Streams

PHP Streams are a unified way of working with file and network resources in PHP. They provide a common interface for reading from and writing to various data sources, abstracting away the differences between files, network sockets, compressed files, and other I/O operations.

## Core Concepts

A stream is referenced using the syntax: `scheme://target`

Common stream wrappers include:
- `file://` - Local filesystem (default)
- `http://`, `https://` - HTTP(S) requests
- `ftp://` - FTP access
- `php://` - Various I/O streams (stdin, stdout, memory, temp)
- `zip://`, `zlib://` - Compressed files
- `data://` - Data URIs

## Why Streams Matter for Large Files

The key advantage is memory efficiency. Instead of loading an entire file into memory at once, streams let you process data in small chunks, making it possible to handle files larger than available RAM.

## Practical Examples for Large File Handling

### 1. Reading Large Files Line by Line

```php
$handle = fopen('large_file.csv', 'r');
if ($handle) {
    while (($line = fgets($handle)) !== false) {
        // Process one line at a time
        processLine($line);
    }
    fclose($handle);
}
```

### 2. Using Stream Contexts for HTTP

```php
$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => 'Authorization: Bearer token123'
    ]
]);

$stream = fopen('https://api.example.com/large-data', 'r', false, $context);
while (!feof($stream)) {
    echo fread($stream, 8192); // Read 8KB chunks
}
fclose($stream);
```

### 3. Copying Large Files Efficiently

```php
// stream_copy_to_stream handles buffering automatically
$source = fopen('large_source.zip', 'r');
$dest = fopen('large_dest.zip', 'w');
stream_copy_to_stream($source, $dest);
fclose($source);
fclose($dest);
```

### 4. Using php://temp for Memory-Efficient Processing

```php
// Automatically switches from memory to temp file if data exceeds 5MB
$temp = fopen('php://temp/maxmemory:5242880', 'r+');
fwrite($temp, $largeData);
rewind($temp);

while (!feof($temp)) {
    $chunk = fread($temp, 8192);
    // Process chunk
}
fclose($temp);
```

### 5. Stream Filters for On-the-Fly Processing

```php
// Read and decompress a gzipped file without loading it all into memory
$handle = fopen('compress.zlib://large_file.gz', 'r');

// Or apply filters to existing streams
$fp = fopen('large_file.txt', 'r');
stream_filter_append($fp, 'string.toupper');
while ($line = fgets($fp)) {
    echo $line; // Automatically converted to uppercase
}
fclose($fp);
```

### 6. Custom Stream Buffer Size

```php
$handle = fopen('huge_file.log', 'r');
// Set 1MB buffer for better performance with large sequential reads
stream_set_read_buffer($handle, 1024  1024);

while (!feof($handle)) {
    $data = fread($handle, 8192);
    processData($data);
}
fclose($handle);
```

## Best Practices

1. Always close streams using `fclose()` to free resources
2. Check for errors - `fopen()` returns `false` on failure
3. Use appropriate chunk sizes - 8KB to 1MB depending on your use case
4. Consider stream filters for transformations instead of loading data into memory
5. Use `php://temp` instead of `php://memory` for potentially large data
6. Leverage `stream_copy_to_stream()` for efficient file copying

Streams are essential for building scalable PHP applications that handle large datasets, file uploads, or API responses without exhausting server memory.