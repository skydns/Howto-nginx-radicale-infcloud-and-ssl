# Howto-nginx-radicale-infcloud-and-ssl
Help for configuration nginx with radicale and infcloud

I searched long for a tutorial, how to configure nginx, radicale and infcloud. While radicale setup is very simple, the configuration of infcloud won't be if you don't know what to do.

Heres the configuration that works fine with me, may be it's altough a solution for you.

Radicale (v2.90) Basic Setup works fine
https://radicale.org/setup/

Letsencrypt works simple and fine too
https://github.com/certbot/certbot
certbot certonly --standalone --rsa-key-size 4096 -d www.yourdomain.net -d yourdomain.net -d dav.yourdomain.net

nginx config sample:
/etc/nginx/sites-available/dav.yourdomain.net.conf

server {
    listen 80;
    listen [::]:80;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /etc/letsencrypt/live/dav.yourdomain.net/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dav.yourdomain.net/privkey.pem;

    server_name dav.yourdomain.net;

    root /var/www/html/infcloud/;

    if ($ssl_protocol = "") {
        return 301 https://$server_name$request_uri;
    }

    ### Radicale Simple WebDAV CalDAV / CardDAV
    location /radicale/ { # The trailing / is important!
    proxy_pass        http://127.0.0.1:5232/; # The / is important!
    proxy_set_header  X-Script-Name /radicale;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass_header Authorization;
    }
}


InfCloud mayor settings is only in the config.js
Download latest here (Version 0.13.1 was the latest as i wrote this help):
https://www.inf-it.com/open-source/clients/infcloud/
search for globalNetworkCheckSettings and change the top like this:

var globalNetworkCheckSettings={
	href: 'https://dav.yourdomain.net/radicale/',
	timeOut: 90000,
	lockTimeOut: 10000,
	checkContentType: true,
	settingsAccount: true,
	delegation: true,
	additionalResources: [],
	hrefLabel: null,
	forceReadOnly: null,
	ignoreAlarms: false,
	backgroundCalendars: []
}


thats all.

