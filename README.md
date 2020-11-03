<!-- Prettier-ignore -->

# Apache-setup

## Directory

-   [Installation](#installation)

-   [Htaccess](#htaccess)

    -   [Deny Access](#deny-access)

    -   [Wordpress](#wordpress)

    -   [Rewrites](#rewrites)

## Installation

```bash
sudo apt-get install apache2
```

or install a LAMP Stack:

```bash
sudo apt-get install apache2 libapache2-mod-php php php-mysql mysql-server
```

<https://wiki.ubuntuusers.de/LAMP/>

**libapache2-mod-php** is providing the PHP-Module for the apache2-webserver

**php-mysql**

## Starting the Server

```bash
sudo systemctl start apache2
```

Once started the server will allways run when the pc is running

## Content

The content of the server will be saved in the "document root". wich depending on the apache and ubuntu version is:

`/var/www/html/`

## Initialize phpinfo

create the **phpinfo.php** in the "document root" and insert:

```php
<?php
phpinfo();
?>
```

insert `http://localhost/phpinfo` into the browser to see all information about the PHP Version and Settings.

Delete this file to make sure Outsiders can not inspect your settings.

## Change Owner

```bash
sudo chown -R www-data:www-data /var/www
```

## 000-default.conf

> /etc/apache2/sites-available/000-default.conf

```config
<Directory  /var/www/html>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
```

<kbd>alt</kbd> + <kbd>t</kbd>

## htaccess

### Deny Access

```apacheconf
FilesMatch "\.(cms|json|log)$">
Order allow,deny
deny from all
</FilesMatch>
```

logs down for the entire server.

_If you find any files with .cms , .json , .log , deny access_

### Wordpress

<https://www.youtube.com/watch?v=V91ffS0yiN0&ab_channel=RankYa>

The .htaccess is a distributed configuration file, and is how Apache handles configuration changes on a per-directory basis.

WordPress uses this file to manipulate how Apache serves files from its root directory, and subdirectories thereof. Most notably, WP modifies this file to be able to handle pretty permalinks.

create a .htaccess file inside the root folder of your wordpress dir

```apacheconf
# BEGIN WordPress

RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

# END WordPress
```

or if your site uses https:

```apacheconf
<IfModlue mod_rewrite.c>
# BEGIN WordPress

RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModlue>

# END WordPress
```

Used for wordpress permalink access

#### Redirect

e.g.

```dot
Redirect 301 /shoop/cart/ https://wplearninglab.com/cart/
```

`Redirect <status> <path past the domain name> <where to redirect to>`

##### redirect for all request made to http

```apacheconf
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]
```

#### Charset

```apacheconf
AddDefaultCharste utf-8
```

#### Keep Header alive

keep alive same connect request instead of resending each HTTP requests

```apacheconf
<ifModule mod_headers.c>
Header set Connection keep-alive
</ifModlue>
```

### Cross Origin

> Important for using Google Fonts , empeded Images/Videos , stylesheets, scripts iframes and some Plugins etc.

##### Cross Oringin Headers

```apacheconf
<IfModule mod_hearders.c>
    Header set Access-Control-Allow-origin "*"
</IfModule>
```

##### Cross Oringin Images

send the CORS headers for images when browsing request ist.

```apacheconf
<IfModule mod_setenvif.c>
    <IfModule mod_headers.c>
        <FilesMatch "\.(bmp|cur|gif|ico|jpe?g|png|svgz?|webp)$">
            setEnvIf Origin ":" IS_CORS
            header set Access-Control-Allow-Origin "*" env=IS_CORS
        </FilesMatch>
    </IfModule>
</IfModule>
```

*allows the server to access these files form an external domain*

##### Cross Origin Web Fonds

```apacheconf
<IfModule mod_headers.c>
    <FilesMatch "\.(eot|otf|tt[cf]|woff2?)$">
        Header set Access-Control-Allow-Origin "*"
    </FilesMatch>
</IfModule>
```

_Allow cross-origin access to web fonts._

##### Errors

```apacheconf
ErrorDocument 404 /404.html
```

_Customize what Apache returns to the client in case of an error._

##### Error prevention

```apacheconf
Options -MultiViews
```

_Disable the pattern matching based on filenames._
_This setting prevents Apache from returning a 404 error as the result of a_
_rewrite when the directory with the same name does not exist._

##### Document Modes

```apacheconf
<IfModule mod_headers.c>
    Header always set X-UA-Compatible "IE=edge" "expr=%{CONTENT_TYPE} =~ m#text/html#i"
</IfModule>
```

_Forces IE to use there latest engine_
_Force Internet Explorer 8/9/10 to render pages in the highest mode_
_available in various cases when it may not._

##### Media Types

*Serve resources with the proper media types (f.k.a.[^1] MIME types [^2]).*
*File extension does not necessary tell you the real file type*
*Media type does*
<br>
*Most moddern **Web Hosts** will have this in there server configs though*

```apacheconf
<IfModule mod_mime.c>

  # Data interchange

    AddType application/atom+xml                        atom
    AddType application/json                            json map topojson
    AddType application/ld+json                         jsonld
    AddType application/rss+xml                         rss
    AddType application/geo+json                        geojson
    AddType application/rdf+xml                         rdf
    AddType application/xml                             xml


  # JavaScript

    # Servers should use text/javascript for JavaScript resources.
    # https://html.spec.whatwg.org/multipage/scripting.html#scriptingLanguages

    AddType text/javascript                             js mjs


  # Manifest files

    AddType application/manifest+json                   webmanifest
    AddType application/x-web-app-manifest+json         webapp
    AddType text/cache-manifest                         appcache


  # Media files

    AddType audio/mp4                                   f4a f4b m4a
    AddType audio/ogg                                   oga ogg opus
    AddType image/bmp                                   bmp
    AddType image/svg+xml                               svg svgz
    AddType image/webp                                  webp
    AddType video/mp4                                   f4v f4p m4v mp4
    AddType video/ogg                                   ogv
    AddType video/webm                                  webm
    AddType video/x-flv                                 flv

    # Serving `.ico` image files with a different media type prevents
    # Internet Explorer from displaying them as images:
    # https://github.com/h5bp/html5-boilerplate/commit/37b5fec090d00f38de64b591bcddcb205aadf8ee

    AddType image/x-icon                                cur ico


  # WebAssembly

    AddType application/wasm                            wasm


  # Web fonts

    AddType font/woff                                   woff
    AddType font/woff2                                  woff2
    AddType application/vnd.ms-fontobject               eot
    AddType font/ttf                                    ttf
    AddType font/collection                             ttc
    AddType font/otf                                    otf


  # Other

    AddType application/octet-stream                    safariextz
    AddType application/x-bb-appworld                   bbaw
    AddType application/x-chrome-extension              crx
    AddType application/x-opera-extension               oex
    AddType application/x-xpinstall                     xpi
    AddType text/calendar                               ics
    AddType text/markdown                               markdown md
    AddType text/vcard                                  vcard vcf
    AddType text/vnd.rim.location.xloc                  xloc
    AddType text/vtt                                    vtt
    AddType text/x-component                            htc

</IfModule>
```

##### Character encodings

```apacheconf
AddDefaultCharset utf-8
```

*Serve all resources labeled as `text/html` or `text/plain` with the media type*
*`charset` parameter set to `UTF-8`.*

```apacheconf
<IfModule mod_mime.c>
    AddCharset utf-8 .appcache \
                     .bbaw \
                     .css \
                     .htc \
                     .ics \
                     .js \
                     .json \
                     .manifest \
                     .map \
                     .markdown \
                     .md \
                     .mjs \
                     .topojson \
                     .vtt \
                     .vcard \
                     .vcf \
                     .webmanifest \
                     .xloc
</IfModule>
```

*Serve the following file types with the media type `charset` parameter set to*
*`UTF-8`.*

### Rewrites

**Rewrite engine**

(1) Turn on the rewrite engine (this is necessary in order for the
`RewriteRule` directives to work).

(2) Enable the `FollowSymLinks` option if it isn't already.

(3) If your web host doesn't allow the `FollowSymlinks` option, you need to
comment it out or remove it, and then uncomment the
`Options +SymLinksIfOwnerMatch` line (4), but be aware of the performance
impact.

(4) Some cloud hosting services will require you set `RewriteBase`.

(5) Depending on how your server is set up, you may also need to use the
`RewriteOptions` directive to enable some options for the rewrite engine.

```apacheconf
<IfModule mod_rewrite.c>

    # (1)
    RewriteEngine On

    # (2)
    Options +FollowSymlinks

    # (3)
    # Options +SymLinksIfOwnerMatch

    # (4)
    # RewriteBase /

    # (5)
    # RewriteOptions <options>

</IfModule>
```

*Uncomment to activate Option*

#### Forcing `https://`

Redirect from the `http://` to the `https://` version of the URL.

```apacheconf
 <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    # (1)
    # RewriteCond %{REQUEST_URI} !^/\.well-known/acme-challenge/
    # RewriteCond %{REQUEST_URI} !^/\.well-known/cpanel-dcv/[\w-]+$
    # RewriteCond %{REQUEST_URI} !^/\.well-known/pki-validation/[A-F0-9]{32}\.txt(?:\ Comodo\ DCV)?$
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
 </IfModule>
```

_(1) If you're using cPanel AutoSSL or the Let's Encrypt webroot method it_
_will fail to validate the certificate if validation requests are redirected to HTTPS. Turn on the condition(s) you need._

#### Suppressing the `www.` at the beginning of URLs

Rewrite www.example.com → example.com

The same content should never be available under two different URLs,
especially not with and without `www.` at the beginning.
This can cause SEO problems (duplicate content), and therefore, you should
choose one of the alternatives and redirect the other one.

(!) NEVER USE BOTH WWW-RELATED RULES AT THE SAME TIME!

(1) Set %{ENV:PROTO} variable, to allow rewrites to redirect with the
appropriate schema automatically (http or https).

(2) The rule assumes by default that both HTTP and HTTPS environments are
available for redirection.
If your SSL certificate could not handle one of the domains used during
redirection, you should turn the condition on.

```apacheconf
<IfModule mod_rewrite.c>

    RewriteEngine On

    # (1)
    RewriteCond %{HTTPS} =on
    RewriteRule ^ - [E=PROTO:https]
    RewriteCond %{HTTPS} !=on
    RewriteRule ^ - [E=PROTO:http]

    # (2)
    # RewriteCond %{HTTPS} !=on

    RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
    RewriteRule ^ %{ENV:PROTO}://%1%{REQUEST_URI} [R=301,L]

</IfModule>
```

### Forcing the `www.` at the beginning of URLs

Rewrite example.com → www.example.com

The same content should never be available under two different URLs,
especially not with and without `www.` at the beginning.
This can cause SEO problems (duplicate content), and therefore, you should
choose one of the alternatives and redirect the other one.

(!) NEVER USE BOTH WWW-RELATED RULES AT THE SAME TIME!

(1) Set %{ENV:PROTO} variable, to allow rewrites to redirect with the
appropriate schema automatically (http or https).

(2) The rule assumes by default that both HTTP and HTTPS environments are
available for redirection.
If your SSL certificate could not handle one of the domains used during
redirection, you should turn the condition on.

Be aware that the following might not be a good idea if you use "real"
subdomains for certain parts of your website.

```apacheconf
<IfModule mod_rewrite.c>

     RewriteEngine On

     # (1)
     RewriteCond %{HTTPS} =on
     RewriteRule ^ - [E=PROTO:https]
     RewriteCond %{HTTPS} !=on
     RewriteRule ^ - [E=PROTO:http]

     # (2)
     # RewriteCond %{HTTPS} !=on

     RewriteCond %{HTTP_HOST} !^www\. [NC]
     RewriteCond %{SERVER_ADDR} !=127.0.0.1
     RewriteCond %{SERVER_ADDR} !=::1
     RewriteRule ^ %{ENV:PROTO}://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

 </IfModule>
```

### SECURITY

#### File Access

Block access to directories without a default document.

You should leave the following uncommented, as you shouldn't allow anyone to
surf through every directory on your server (which may include rather
private places such as the CMS's directories).

```apacheconf
<IfModule mod_autoindex.c>
    Options -Indexes
</IfModule>
```

---

Block access to all hidden files and directories except for the
visible content from within the `/.well-known/` hidden directory.

These types of files usually contain user preferences or the preserved state
of a utility, and can include rather private places like, for example, the
`.git` or `.svn` directories.

The `/.well-known/` directory represents the standard (RFC 5785) path prefix
for "well-known locations" (e.g.: `/.well-known/manifest.json`,
`/.well-known/keybase.txt`), and therefore, access to its visible content
should not be blocked.

```apacheconf
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_URI} "!(^|/)\.well-known/([^./]+./?)+$" [NC]
    RewriteCond %{SCRIPT_FILENAME} -d [OR]
    RewriteCond %{SCRIPT_FILENAME} -f
    RewriteRule "(^|/)\." - [F]
</IfModule>
```

## WEB PERFORMANCE

### Compression

compress files before they are send down to client

```apacheconf
<IfModule mod_deflate.c>

    # Force compression for mangled `Accept-Encoding` request headers
    #
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding
    # https://calendar.perfplanet.com/2010/pushing-beyond-gzipping/

    <IfModule mod_setenvif.c>
        <IfModule mod_headers.c>
            SetEnvIfNoCase ^(Accept-EncodXng|X-cept-Encoding|X{15}|~{15}|-{15})$ ^((gzip|deflate)\s*,?\s*)+|[X~-]{4,13}$ HAVE_Accept-Encoding
            RequestHeader append Accept-Encoding "gzip,deflate" env=HAVE_Accept-Encoding
        </IfModule>
    </IfModule>

    # - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

    # Compress all output labeled with one of the following media types.
    #
    # https://httpd.apache.org/docs/current/mod/mod_filter.html#addoutputfilterbytype

    <IfModule mod_filter.c>
        AddOutputFilterByType DEFLATE "application/atom+xml" \
                                      "application/javascript" \
                                      "application/json" \
                                      "application/ld+json" \
                                      "application/manifest+json" \
                                      "application/rdf+xml" \
                                      "application/rss+xml" \
                                      "application/schema+json" \
                                      "application/geo+json" \
                                      "application/vnd.ms-fontobject" \
                                      "application/wasm" \
                                      "application/x-font-ttf" \
                                      "application/x-javascript" \
                                      "application/x-web-app-manifest+json" \
                                      "application/xhtml+xml" \
                                      "application/xml" \
                                      "font/eot" \
                                      "font/opentype" \
                                      "font/otf" \
                                      "font/ttf" \
                                      "image/bmp" \
                                      "image/svg+xml" \
                                      "image/vnd.microsoft.icon" \
                                      "text/cache-manifest" \
                                      "text/calendar" \
                                      "text/css" \
                                      "text/html" \
                                      "text/javascript" \
                                      "text/plain" \
                                      "text/markdown" \
                                      "text/vcard" \
                                      "text/vnd.rim.location.xloc" \
                                      "text/vtt" \
                                      "text/x-component" \
                                      "text/x-cross-domain-policy" \
                                      "text/xml"

    </IfModule>

    # - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

    # Map the following filename extensions to the specified encoding type in
    # order to make Apache serve the file types with the appropriate
    # `Content-Encoding` response header (do note that this will NOT make
    # Apache compress them!).
    #
    # If these files types would be served without an appropriate
    # `Content-Encoding` response header, client applications (e.g.: browsers)
    # wouldn't know that they first need to uncompress the response, and thus,
    # wouldn't be able to understand the content.
    #
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding
    # https://httpd.apache.org/docs/current/mod/mod_mime.html#addencoding

    <IfModule mod_mime.c>
        AddEncoding gzip              svgz
    </IfModule>

</IfModule>
```

### Cache expiration

*How long is the server supposed to cache a file*

Serve resources with a far-future expiration date.

(!) If you don't control versioning with filename-based cache busting, you
should consider lowering the cache times to something like one week.

```apacheconf
<IfModule mod_expires.c>

    ExpiresActive on
    ExpiresDefault                                      "access plus 1 month"

  # CSS

    ExpiresByType text/css                              "access plus 1 year"


  # Data interchange

    ExpiresByType application/atom+xml                  "access plus 1 hour"
    ExpiresByType application/rdf+xml                   "access plus 1 hour"
    ExpiresByType application/rss+xml                   "access plus 1 hour"

    ExpiresByType application/json                      "access plus 0 seconds"
    ExpiresByType application/ld+json                   "access plus 0 seconds"
    ExpiresByType application/schema+json               "access plus 0 seconds"
    ExpiresByType application/geo+json                  "access plus 0 seconds"
    ExpiresByType application/xml                       "access plus 0 seconds"
    ExpiresByType text/calendar                         "access plus 0 seconds"
    ExpiresByType text/xml                              "access plus 0 seconds"


  # Favicon (cannot be renamed!) and cursor images

    ExpiresByType image/vnd.microsoft.icon              "access plus 1 week"
    ExpiresByType image/x-icon                          "access plus 1 week"

  # HTML

    ExpiresByType text/html                             "access plus 0 seconds"


  # JavaScript

    ExpiresByType application/javascript                "access plus 1 year"
    ExpiresByType application/x-javascript              "access plus 1 year"
    ExpiresByType text/javascript                       "access plus 1 year"


  # Manifest files

    ExpiresByType application/manifest+json             "access plus 1 week"
    ExpiresByType application/x-web-app-manifest+json   "access plus 0 seconds"
    ExpiresByType text/cache-manifest                   "access plus 0 seconds"


  # Markdown

    ExpiresByType text/markdown                         "access plus 0 seconds"


  # Media files

    ExpiresByType audio/ogg                             "access plus 1 month"
    ExpiresByType image/apng                            "access plus 1 month"
    ExpiresByType image/bmp                             "access plus 1 month"
    ExpiresByType image/gif                             "access plus 1 month"
    ExpiresByType image/jpeg                            "access plus 1 month"
    ExpiresByType image/png                             "access plus 1 month"
    ExpiresByType image/svg+xml                         "access plus 1 month"
    ExpiresByType image/webp                            "access plus 1 month"
    ExpiresByType video/mp4                             "access plus 1 month"
    ExpiresByType video/ogg                             "access plus 1 month"
    ExpiresByType video/webm                            "access plus 1 month"


  # WebAssembly

    ExpiresByType application/wasm                      "access plus 1 year"


  # Web fonts

    # Collection
    ExpiresByType font/collection                       "access plus 1 month"

    # Embedded OpenType (EOT)
    ExpiresByType application/vnd.ms-fontobject         "access plus 1 month"
    ExpiresByType font/eot                              "access plus 1 month"

    # OpenType
    ExpiresByType font/opentype                         "access plus 1 month"
    ExpiresByType font/otf                              "access plus 1 month"

    # TrueType
    ExpiresByType application/x-font-ttf                "access plus 1 month"
    ExpiresByType font/ttf                              "access plus 1 month"

    # Web Open Font Format (WOFF) 1.0
    ExpiresByType application/font-woff                 "access plus 1 month"
    ExpiresByType application/x-font-woff               "access plus 1 month"
    ExpiresByType font/woff                             "access plus 1 month"

    # Web Open Font Format (WOFF) 2.0
    ExpiresByType application/font-woff2                "access plus 1 month"
    ExpiresByType font/woff2                            "access plus 1 month"


  # Other

    ExpiresByType text/x-cross-domain-policy            "access plus 1 week"

</IfModule>
```

---

## Tipps

### RedirectMatch

```apacheconf
RedirectMatch 403 ^/protected/?$
```

*gives out 403 when trying to use the url*


### Redirect to Download

```apacheconf
RedirectMatch 301 /download/theme https://somelink-to-download
```

*redirect from the "pretty path" to the actual download*

### Redirect 404 with `<base>`

```apacheconf
ErrorDocument 404 /404/
```

*When entering a non existent URL, Visitor will stay on that URL but the /404/ dir page will be displayed.*

*However linked content, like pictures on this page will fail as the URL is in fact wrong*

```html
<head>
  <base href="https://www.mysite.com/404/">
</head>
```

*Adding a Base Tag to this Page can circumvent this issue by giving a base URL*

*Base URL informs the browser where this page resides and where it should based all the path for all the files from (relative URL)*

```html
<head>
  <base href="auto-detect-on-publish">
</head>
```

***auto-detect-on-publish** is also possible???*

### www-data

#### Add User to www-data Group

For being able to login via FTP , edit file , upload files , create Folders and what not...

```bash
sudo usermod -a -G www-data <User>
```

confirm `<User>` is part of www-data Group

```bash
groups <User>
```

## Links

htaccess

**html5 boilerplate**
<https://github.com/h5bp/html5-boilerplate>
_dist/.htaccess_

**Weaver Tips**
<https://weaver.tips/tag/htacess>

**htaccess tester**
<https://htaccess.madewithlove.be/>

[^1] _formally known as_

[^2] A **media type** (also known as a **Multipurpose Internet Mail Extensions** or MIME type) is a standard that indicates the nature and format of a document, file, or assortment of bytes. It is defined and standardized in IETF's RFC 6838.

The simplest MIME type consists of a type and a subtype; these are each strings which, when concatenated with a slash (/) between them, comprise a MIME type. No whitespace is allowed in a MIME type:

> `type/subtype`

<https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types>
