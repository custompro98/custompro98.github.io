---
layout: post
title: Deploying Laravel
categories:
- laravel
tags:
- fullstack
- laravel
- php
---

Last post, I wrote about [Laravel](https://laravel.com/) and building [onetimething](https://onetimething.custompro98.club) to get a feel for what it was like to actually make something with Laravel. Now that it was built, I had to learn how to get it out there. Like any recovering JS engineer, I looked around for the "Vercel of Laravel" and found [Laravel Forge](https://forge.laravel.com) and [Laravel Vapor](https://vapor.laravel.com). Be still my beating heart, two first party options for deployments?? Turns out PHP developers expect real products from their community, not toy-apps to show off the latest stack. Reasonable. Then it hit me - I have a cloud VM, I've deployed other applications there in the past. Time to roll up my sleeves and remember how to manage a server.

Between serverless for side projects and containers on Kubernetes for work, it had been a long time since I actually managed my own hardware. Let's get into it.

_Note: this set of anecdotal instructions assumes you already have a cloud VM with networking attached and Caddy installed and running as a service and is otherwise DEFINITELY incomplete_

1. Good artists steal, reference [this blog post](https://jorgeglz.io/blog/setting-up-laravel-applications-with-caddy-2/) for setting up Caddy to work with Laravel.
2. onetimething was built against PHP 8.3, [avoid any version problems](https://www.linuxtuto.com/how-to-install-php-8-3-on-ubuntu-22-04/)
3. Make sure to follow the steps to `update-alternatives --config php`
4. Install the necessary PHP extensions, `sudo apt install php8.3-mysql php8.3-imap php8.3-ldap php8.3-xml php8.3-curl php8.3-mbstring php8.3-zip php8.3-sqlite3` (disclaimer: I don't actually know which of these I'm using other than sqlite3)
5. Git clone my repository into `/var/wwww/<application_name>`
6. `cd /var/www/<application_name>` and run `sudo chown -R www-data:www-data ./database` and `sudo chown -R www-data:www-data ./storage`
7. `sudo composer install` (note: it's probably better to not run this as sudo)
8. Restart Caddy with `sudo service caddy reload`
9. Configure a DNS A record to point at your (sub-)domain to the public IP address of your cloud VM
10. Profit (just a figure of speech in this case)
