---
title: "Hosting the CyberDiscord Open"
date:   2023-06-11
categories:
  - CyberDiscord
tags:
  - competitions
  - cyberdiscord
  - thoughts
---

Last week, we organized and hosted the first-ever CyberDiscord Open, a cybersecurity competition that brought together 118 teams from five different countries: the United States, Canada, United Kingdom, South Korea, and Japan! We managed to plan and execute the entire competition in just over three weeks, so I wanted to talk about our goals, the challenges we faced, the solutions we implemented, and some valuable lessons we learned along the way. 

## Goals

Drawing from my past experience as a CyberPatriot competitor, I determined some elements that should be present in the competition.

### No Age, Location, or Education Requirements

In competitions like CyberPatriot and CCDC, teams are typically restricted to students from the same school. As a previous competitor, I always wished for the opportunity to compete with my friends. We wanted competitors to be able to compete with anyone they wanted, regardless of age or geographical location.

### No Hardware Requirements

Many competitors don't have personal computers powerful enough to run images locally, or face other limitations such as limited storage space or unreliable network connections. Issues like these make it difficult to teach new people, and we wanted to make the competition as accessible as possible.

<figure>
<img src="https://i.imgur.com/p1LG4EP.png">
<figcaption>It's really important to be able to train all competitors.</figcaption>
</figure>

### No VPNs

In competitions like [Hivestorm](https://hivestorm.org/) that host online virtual machines, competitors are required to connect to a VPN to access the challenges. Users always encounter problems with the VPN and supporting them is a pain. Any solution that eliminated the need for a VPN would simplify the process for both competitors and organizers.

## The Solution

Hosting images online seemed like the most viable solution, as it meant that teams could collaborate on images without losing their progress anytime someone new wanted to hop on an image. Furthermore, competitors who lacked the necessary resources to run images locally could still participate. 

In order to achieve this, we created a custom portal that let teams access a web console without the need for a VPN. While we had a backup plan to distribute local images in case of any issues, we were fortunate enough to not encounter any problems.

<figure>
<img src="https://i.imgur.com/tsDqQcP.png">
<figcaption>Coincidence that after multiple all-nighters, the last image zipped up had an md5sum starting with bed?</figcaption>
</figure>

Initially, we had two ESXi servers that we planned to use for this competition, but it quickly became evident that it wouldn't be enough. We ended up using eleven ESXi servers: nine dedicated to hosting virtual machines and two for operations. Collectively, these servers had a combined 680 cores.

<figure>
<img src="https://i.imgur.com/fe02VBV.png">
<figcaption>Maximum usage was about 73% for one of the larger servers.</figcaption>
</figure>

## Challenge Development

Our goal for the challenges was to strike a balance between being beginner-friendly while still presenting a significant challenge to experienced teams. I think we did a great job in achieving this.

We initially wanted to do a server core image, however we quickly realized that without a VPN, users wouldn't have access to remote tools like WAC or RSAT to manage the server core. We swiftly abandoned that idea in favor of Server 2022, but we'll continue exploring better support for operating systems with no GUI for the future.

Each team was provided four virtual machines: Windows 10, Fedora 36, Server 2022, and Ubuntu 22, as well as a RSA Cryptography challenge. Both Windows images were assigned two cores (because a certain goose friend liked to use a lot of CPU) and both Linux images were allocated one core each. 

In hindsight, it was probably better to have given two cores to Fedora instead of Windows 10. We discovered pretty quickly that Cinnamon liked to use _a lot_ of CPU.

<figure>
<img src="https://i.imgur.com/D1gbEQ1.png">
<figcaption>Next year, we'll probably use a different flavor of Fedora.</figcaption>
</figure>

## Some Commonly Missed Vulnerabilities

These are some of the most frequently missed vulnerabilities across all the images. While we won't be posting `answers.zip`, I hope this will help teams in preparations for future events.

### <u>Windows 10 - Highest Score: 83 points</u>

### Thunderbird Do Not Track header is enabled

In Thunderbird, go to Preferences, Privacy & Security, Privacy, Web Content. Click the checkbox for Send websites a "Do Not Track" signal that you don't want to be tracked.

![thunderbird](https://i.imgur.com/MKw5Dce.png)

### SSH public key added for ballen

You were given ballen's public and private keys and asked to configure public key authentication. If you read the Microsoft documentation for [Key-based authentication](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement) under the "Administrative user" section:

> The contents of your public key (\.ssh\id_ed25519.pub) needs to be placed on the server into a text file called `administrators_authorized_keys` in C:\ProgramData\ssh\. You can copy your public key using the OpenSSH scp secure file-transfer utility, or using a PowerShell to write the key to the file. The ACL on this file needs to be configured to only allow access to administrators and System.

First, we should ensure that the contents of our public key are located in `C:\ProgramData\ssh\administrators_authorized_keys`. We can do that with the following command:

```ps
Get-Content -Path "C:\Users\ballen\Documents\id_rsa.pub" | Add-Content -Force -Path "C:\ProgramData\ssh\administrators_authorized_keys"
```

Next, make sure that the proper permissions are set on the file, to only allow access to Administrators and System:

```ps
icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

Finally, ensure that `PubkeyAuthentication no` is not configured in `C:\ProgramData\ssh\sshd_config`.

### <u>Fedora 36 - Highest Score: 33 points</u>

### Insecure permissions on MySQL database directory fixed

Everyone should not be able to read or execute in the MySQL directory, and user and group ownership should be given to the `mysql` group and `mysql` user.

```bash
[viktor@piltover ~]$ ls -ld /var/lib/mysql
drwxr-xr-x 8 mysql mysql 4096 Jun 11 01:33 /var/lib/mysql
[viktor@piltover ~]$ sudo chmod 750 /var/lib/mysql
[viktor@piltover ~]$ 
```

### Apache does not allow overrides on the root directory

The `AllowOverride` directive determines what configuration directives can be overridden or changed by files located in the web server's directory. To disable this, navigate to the Apache configuration at `/etc/httpd/conf/httpd.conf` and set `AllowOverride` on the `/` directory to `None`.

```bash
<Directory />
    AllowOverride None
    Require all denied
</Directory>
```

### <u>Server 2022 - Highest Score: 86 points</u>

### DNS service restarts after failure

The DNS server was a critical service on the machine and we should make sure that it restarts if it ever crashes. Open Windows Services (services.msc) and select the DNS Server. Go to Properties, Recovery, and choose "Restart the Service" after each failure.

![dns](https://i.imgur.com/nO9Bb8N.png)

### Authenticated users cannot issue/manage certs

Run certsrv.msc. This will open the Certification Authority. Navigate to Properties and Security, then unselect "Issue and Manage Certificates" and "Manage CA" for Authenticated Users.

![adcs](https://i.imgur.com/mVVeIDF.png)

### <u>Ubuntu 22 - Highest Score: 77 points</u>

### Role-based access control methods enabled for MongoDB

We should require clients to authenticate themselves as valid users, thereby restricting their actions to the assigned roles. Modify the MongoDB configuration file at `/etc/mongod.conf` by adding the following lines:

```bash
security:
  authorization: "enabled"
```

Then, save the file and run `sudo service mongodb restart`. 

### Inline scripts are not allowed by the Nginx content security policy

In our Nginx configuration, the `"script-src"` directive guards the loading and execution of JavaScript. However, using `"script-src 'unsafe-inline'"` allows unknown scripts to be executed on the site. If you audit the default site located at `/etc/nginx/sites-available/default`, you will notice the unsafe configuration:

```bash
add_header Content-Security-Policy "script-src 'unsafe-inline';";
```

Simply remove `'unsafe-inline'` and save the file. [Click here](https://content-security-policy.com/unsafe-inline/) to learn more about the `unsafe-inline` keyword.

## Wrapping Up

The first CyberDiscord Open proved to be a resounding success! Congrats to Team013, ඞඞඞ for the victory, and shoutouts to everyone who competed. To Dwayne, Keith, Landon, and Akshay - thank you for your invaluable contributions. This competition wouldn't have been possible without you. Additionally, thank you to MSI for their generous sponsorship. I'm looking forward to hosting season two next year!
