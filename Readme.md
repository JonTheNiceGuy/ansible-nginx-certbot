# JonTheNiceGuy's Ansible Role for configuring nginx

This role is a stand-alone role, and is rather opinionated :)

It creates a "catch-all" HTTP-only default server, which ONLY upgrades connections to HTTPS and handles certbot domain validation.

After that, it parses the following variables:

* [`wordpress_sites`](#Wordpress-Sites)
* [`basic_sites`](#Basic-Sites)
* [`proxy_sites`](#Proxy-Sites) (for things like, docker containers, local binaries, or reverse proxying to internal services)
* [`redirect_sites`](#Redirect-Sites) (for things where you might want to redirect `old.site.com` to `new.site.com` or `www.site.com` to `site.com`)

This role does not support any non-HTTPS holding directories.

## Wordpress Sites

This does not create wordpress sites, it just puts the right config into nginx for this!

In your vars passed to the role, or in the `host_vars` or `group_vars` for your web server, include a customized version of this variable:

```yaml
wordpress_sites:
  "wordpress.example.com":     # This is the primary name of the site. The following values are the defaults, except for alts where an example
                               # is specified.
    # alts:                    # This is a list of all the aliases for these sites. The following is an example!
    # - wp.example.com           # This can include any other DNS name. Note this may impact Google status!
                                 # You should use redirects instead!
    # www_prefix: true         # Default value. Instructs Certbot to mint a certificate for the the prefix www.<sitename> plus any alts.
                               # If set to false, any alt names where www is required must be individually specified.
    # pretty_urls: true        # Default value, tries either the file for the url first, or failing that trying to run
                               # /index.php?<url_path>
    # webroot: /var/www/wordpress.example.com
                               # This is the default, based on tasks/main.yml
    # locations: []            # These are treated as extra wordpress directories.
    # includes: ''             # If there are any extra files created for this site, this is where to include them, either
                               # as a string, or as an array of includes.
    # max_body_size: 50G       # This is the default value. Override it (higher or lower) if required.
    # body_buffer_size: 25M    # This is the default value. Override it (higher or lower) if required.
    # enable_letsencrypt: true # This is default to true. If set to false, specify tls_certificate and tls_certificate_key
                               # or default to using the "snake-oil-certs".
    # mozilla_config: intermediate
                               # This can be set to intermediate (default), modern or old. It is based on the values from
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=old&openssl=1.1.1d&guideline=5.4
    # HSTS: false              # Default value. Set to false as this may cause issues if you have problems with recertifying TLS in the future!
```

## Basic Sites

This is for "simple" PHP applications or static ones. Basically, anything which is not wordpress, and requires little-to-no customization in the nginx path!

In your vars passed to the role, or in the `host_vars` or `group_vars` for your web server, include this variable:

```yaml
basic_sites:
  "static.example.com":        # This is the primary name of the site. The following values are the defaults, except for alts where an example
                               # is specified.
    # alts:                    # This is a list of all the aliases for these sites. The following is an example!
    # - my.example.com           # This can include any other DNS name. Note this may impact Google status!
                                 # You should use redirects instead!
    # www_prefix: true         # Default value. Instructs Certbot to mint a certificate for the the prefix www.<sitename> plus a prefix of www. for all alts.
                               # If set to false, any alt names where www is required must be individually specified.
    # pretty_urls: true        # Default value, tries either the file for the url first, or failing that trying to run
                               # /index.php?<url_path>
    # webroot: /var/www/static.example.com
                               # This is the default, based on tasks/main.yml
    # locations: []            # These are treated as extra wordpress directories.
    # includes: ''             # If there are any extra files created for this site, this is where to include them, either
                               # as a string, or as an array of includes.
    # max_body_size: 50G       # This is the default value. Override it (higher or lower) if required.
    # body_buffer_size: 25M    # This is the default value. Override it (higher or lower) if required.
    # enable_letsencrypt: true # This is default to true. If set to false, specify tls_certificate and tls_certificate_key
                               # or default to using the "snake-oil-certs".
    # mozilla_config: intermediate
                               # This can be set to intermediate (default), modern or old. It is based on the values from
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=old&openssl=1.1.1d&guideline=5.4
    # HSTS: false              # Default value. Set to false as this may cause issues if you have problems with recertifying TLS in the future!
```

## Proxy Sites

If you have some sort of service running in something like Docker or you want to forward an HTTPS connection to an HTTP service inside your network, this is the
config section for you!

In your vars passed to the role, or in the `host_vars` or `group_vars` for your web server, include this variable:

```yaml
proxy_sites:
  "someservice.example.com":   # This is the primary name of the site.
    target: http://192.0.2.1:80
                               # This value is mandatory. It points to the web service you are connecting to.

                               # The following values are the defaults, except for alts where an example
                               # is specified.
    # forward_host_header: true
                               # This forwards the http_host and host values to the upstream service. Note this may
                               # cause issues with some services!
    # alts:                    # This is a list of all the aliases for these sites. The following is an example!
    # - svc.example.com          # This can include any other DNS name. Note this may impact Google status!
                                 # You should use redirects instead!
    # www_prefix: true         # Default value. Instructs Certbot to mint a certificate for the the prefix www.<sitename> plus a prefix of www. for all alts.
                               # If set to false, any alt names where www is required must be individually specified.
    # enable_letsencrypt: true # This is default to true. If set to false, specify tls_certificate and tls_certificate_key
                               # or default to using the "snake-oil-certs".
    # mozilla_config: intermediate
                               # This can be set to intermediate (default), modern or old. It is based on the values from
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=old&openssl=1.1.1d&guideline=5.4
    # HSTS: false              # Default value. Set to false as this may cause issues if you have problems with recertifying TLS in the future!
```

## Redirect Sites

More than likely, you want to redirect www.example.com to example.com (or vice versa). This is the config block that will do it.

In your vars passed to the role, or in the `host_vars` or `group_vars` for your web server, include this variable:

```yaml
redirect_sites:
  "oldsite.example.com":           # This is the primary name of the site.
    target_server_name: newsite.example.com
                               # This value is mandatory. It points to the new hostname you are connecting to.

                               # The following values are the defaults.
    # alts: []                 # This is a list of all the aliases for these sites.
    # www_prefix: true         # Default value. Instructs Certbot to mint a certificate for the the prefix www.<sitename> plus a prefix of www. for all alts.
                               # If set to false, any alt names where www is required must be individually specified.
    # enable_letsencrypt: true # This is default to true. If set to false, specify tls_certificate and tls_certificate_key
                               # or default to using the "snake-oil-certs".
    # mozilla_config: intermediate
                               # This can be set to intermediate (default), modern or old. It is based on the values from
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1d&guideline=5.4
                               # https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=old&openssl=1.1.1d&guideline=5.4
    # HSTS: false              # Default value. Set to false as this may cause issues if you have problems with recertifying TLS in the future!
```
