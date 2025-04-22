---
title: "Upcoming Changes to eCitadel"
date:   2025-04-22
categories:
  - eCitadel
tags:
  - competitions
  - ecitadel
  - thoughts
---

Let's first recap eCitadel season II, then I'll talk about some of the new changes we're making to eCitadel this year.

## Season II Recap

Season II of the eCitadel Open had 159 teams and 569 competitors worldwide. In case you forgot, here's a quick recap of the environment and challenges:

### Environment

Teams were given a Windows 2016 server, Windows 2022 server, Linux Mint 21 server, and a headless Alma Linux 9 server.

All of the machines were on the same network and domain which made interconnectivity of the machines possible, for example, the two HTTP websites on Alma and Windows 2022 relied on the database on Mint and DNS on Windows 2016 to function.

We required all scored services to be externally available and checked them based on content and functionality. Teams had a pfSense box performing 1:1 Network Address Translation (NAT) and were assigned external IP addresses.

Teams also had two web challenges and two business-style injects to complete.

### Incident Reports

This was the first year we asked teams to submit incident reports. Almost all teams struggled with submitting accurate reports, and no teams received the maximum score. This is what were were looking for when grading incident reports.

Teams were asked to only submit a report for any active connections with an external IP. Many teams reported some malware that was listening or a forensic event that happened in the past. These did not receive any credit because it did not meet the criteria of being an _active, established_ connection.

Incident reports should include the source and destination IP addresses, how the attackers are maintaining persistence, and what you did to address the incident.

There were two chances for incident response. On Windows 2016, there was a reverse HTTP meterpreter, and on Alma Linux 9, there was a Golang backdoor.

### Reverse HTTP meterpreter - 3%

On Windows, you can use a tool like TCPView to view established connections.

![tcpview](https://i.imgur.com/4rbgusV.png)

Even though 19% of teams even found and removed the reverse HTTP meterpreter on Windows, only 3% submitted incident reports for it. Remember to investigate any malware that you do find before you remove it.

<figure>
<img src="https://i.imgur.com/ObOABSR.png">
<figcaption>By the end of competition, there were a total of 322 meterpreter sessions. If your team was one of the few that managed to remove it, congratulations!</figcaption>
</figure>

### Commonly Missed Vulnerabilities

Here are some of the most commonly missed vulnerabilities as well as how to find and solve them across all images.

### <u>Alma Linux 9 - Highest Score: 85 points</u>

### Domain user butler is not an administrator - 23%

If you audit the contents of `/etc/group`, you will notice the following misconfiguration:

```bash
wheel:x:10:sigurd,butler@company.local
```

The `wheel` group on RHEL-based linux systems is the equivalent of the `sudo` group on Debian, and grants administrator privileges. As the domain user `butler` is not an authorized administrator, we should remove him from the `wheel` group.

### IPv4 forwarding has been disabled - 22%

As this linux server is not acting as a router, we do not need to be forwarding packets meant for other destinations (other than ourself).

Edit the file `/etc/sysctl.conf` and change the line that says `net.ipv4.ip_forward=1` to 0. Save the file, then run `sysctl -p` to reload the settings, or reboot.

### Firewall protection has been enabled - 34%

Enabling a host-based firwall is very important to system security. You can view the status of firewalld with the command `sudo systemctl status firewalld`.

![firewalld](https://i.imgur.com/0u3QBGU.png)

Simply type `sudo systemctl enable --now firewalld` to enable and start the firewall.

### DNF automatically installs updates - 20%

You can use a service to automatically download and install any new updates (for example security updates).

Type `sudo dnf install dnf-automatic` to install dnf-automatic. Then edit the configuration file at `/etc/dnf/automatic.conf` and change the following setting:

```bash
apply_updates=yes
```

Finally, run `sudo systemctl enable --now dnf-automatic.timer` to enable and start the systemd timer.

### firewalld has been updated - 47%

Updating installed applications and services to fix security vulnerabilities is an important principle of good cybersecurity. To update firewalld, type `sudo dnf upgrade firewalld`.

### Prohibited software nyancat removed - 20%

If you ever logged in as root, you may have encountered:

![nyancat](https://i.imgur.com/2N9IDgu.png)

You can quickly find how it is being started by looking in the root user's `.bashrc`, which is a file that is executed when a user logs in.

![bashrc](https://i.imgur.com/j3RU85e.png)

Remove the entry from root's `.bashrc` and delete the nyancat binary with `rm -f /usr/sbin/meowmeowmeowmeowmeowmeowmeowmeowmeowmeow`.

### SSH root login has been disabled - 50%

Only authorized employees should be allowed to login to the SSH server.

Open the file at `/etc/ssh/sshd_config` and change the line that says `PermitRootLogin yes` to `PermitRootLogin no`. Save the file and exit, then restart the service with `sudo systemctl restart sshd`.

### SSH does not permit empty passwords - 35%

We should always require a password when logging into the SSH server, as an additional security measure.

Open the file at `/etc/ssh/sshd_config` and change the line that says `PermitEmptyPasswords yes` to `PermitEmptyPasswords no`. Save the file and exit, then restart the service with `sudo systemctl restart sshd`.

### FTP anonymous access is disabled - 41%

Only authorized employees should be allowed to login to the FTP server.

Open the file at `/etc/vsftpd/vsftpd.conf` and change the line that says `anonymous_enable=YES` to `anonymous_enable=NO`. Save the file and exit, then restart the service with `sudo systemctl restart vsftpd`.

### <u>Linux Mint 21 - Highest Score: 80 points</u>

### Forensics Question 2 correct - 53%

### Removed unauthorized MySQL user bees - 35%

This question asked you to identify the password hash of an unauthorized user on the database.

First, login to the database with the command `mysql -u sigurd -p` and enter the credentials you were provided.

MySQL stores information about users in the `user` table of the `mysql` database. You can view all users and their password hashes with the command `SELECT user,password FROM mysql.user;`.

![mysql](https://i.imgur.com/pvTd0BJ.png)

Looking at the output, it is obvious that `bees` is the unauthorized user. After you have answered the question, you can now delete this user with the command `DROP USER bees;`.

### Removed hidden user Admin - 26%

If you audit the contents of `/etc/passwd`, you will notice the following entry:

```bash
Admin:x:99:99:Admin:/bin:/bin/bash
```

This user will not show up in Users and Groups because they have a UID less than 1000 (typically reserved for system accounts), but they can login because they have a valid shell. You can delete them with `sudo userdel Admin`.

### A default minimum password age is set - 41%

Open the `/etc/login.defs` file. Find the line that says `PASS_MIN_DAYS  0` and change it to any value greater than 0.

### Uncomplicated Firewall (UFW) protection has been enabled - 73%

Enabling a host-based firwall is very important to system security. You can view the status of UFW with the command:

```bash
$ sudo ufw status
Status: inactive
```

Simply type `sudo ufw enable` to enable the firewall.

### Removed ICMP backdoor - 0%

If you audit the list of services on the system, you will notice the unauthorized `prism.service`, started with systemd. Further investigation into this service will reveal that it is starting the [Prism backdoor](https://github.com/andreafabrizi/prism){:target="_blank"}.

To remove, simply disable and stop the service, then remove the binary.

```bash
$ sudo systemctl disable --now prism
$ sudo rm -f /usr/sbin/prism
```

### Chromium blocks intrusive advertisements - 8%

Open Chromium and navigate to `chrome://settings/content/ads`. Select **Ads are blocked on sites that show intrusive or misleading ads**.

![chromium](https://i.imgur.com/pRm6AoL.png)

### <u>Windows Server 2016 - Highest Score: 90 points</u>

### Created user account tulipsnake - 64%

Part one of the "New User Inject" requested you to create a new domain user account to be added to the "company.local" domain.

Open the run dialog and type `dsa.msc` to open Active Directory Users and Computers. Click **company.local -> Users** on the left side of the window. Right click on **Users** and select **New -> User**. Enter in `tulipsnake` for **Full name** and **User logon name**, then select **Next >**. Enter in a secure temporary password of your choosing and select **Next >**. Verify the information is correct and select **Finish**.

### User thumper is not an Domain Admin - 39%

Open the run dialog and type `dsa.msc` to open Active Directory Users and Computers. Click **company.local -> Users** on the left side of the window. Double-click on **Domain Admins** to open a Properties window. Navigate to the **Members** tab, select **thumper**, and click **Remove**, then click **OK** to apply the changes and close the Properties window.

### Firewall protection has been enabled - 75%

Open the run dialog and type `wf.msc` to open Windows Firewall with Advanced Security. Click on **Windows Firewall Properties** in the middle of the window. On all three profile tabs, ensure that Firewall state is set to **On (recommended)**.

### Removed reverse HTTP meterpreter - 19%

See [above](#reverse-http-meterpreter---3) for how to find and solve this vulnerability.

### <u>Windows Server 2022 - Highest Score: 76 points</u>

### Forensics Question 2 correct - 43%
### Removed malicious administrative SSH key - 23%

This question asked for the comment in an unauthorized public key on the OpenSSH server.

Public keys are stored in the file `C:\ProgramData\ssh\administrators_authorized_keys`. If you audit the contents of this file, you will find the unauthorized key and its comment.

After you have answered this question, delete the key by removing the line from the file.

### Users may not read the MediaWiki LocalSettings file - 0%

The MediaWiki configuration file is located at `C:\inetpub\wwwroot\LocalSettings.php`. If you audit the file you will notice that the `Users` group has read permissions on the file.

### Windows Defender Firewall service is running - 20%
### Microsoft FTP service has been stopped and disabled - 25%

Open the run dialog and type `services.msc` to open Windows Services. Make sure the Windows Defender Firewall is running and configured to automatically start. Make sure the Microsoft FTP service has been stopped and its startup type set to disabled.

### Windows automatically checks for updates - 27%

Open the run dialog and type `gpedit.msc` to open the Local Group Policy Editor console. Navigate to **Computer Configuration -> Administrative Templates -> Windows Components -> Windows Updates**. Double click on **Configure Automatic Updates**. Enable the setting by selecting **4 - Auto download and schedule the install**. Also, check the box for **Install updates for other Microsoft products**.

### Google Chrome has been updated - 59%

Open Google Chrome and navigate to https://google.com/chrome. Click on the Download Chrome button to download the latest installer and run it.

## Season III

Registration for the 2025 eCitadel Open is now open! This year we're making lots of changes to the competition structure, environment, and scoring.

### Registration Fees

We've always wanted to keep eCitadel free to participate. In the past, we've funded prizes and new hardware out of our own pockets, however that's just not sustainable anymore. As the competition rapidly grows, we need even more hardware and resources to be able to support more teams. For example, we currently only have a single [10 Gigabit switch](https://sg.store.ui.com/sg/en/collections/unifi-switching-enterprise-10-gbps-ethernet/products/usw-enterprisexg-24){:target="_blank"} available to run this competition. Adding any more servers would affect I/O speeds for all teams, which means that we have a maximum of 13 servers for teams (because we physically ran out of ports on the switch).

<figure>
<img src="https://i.imgur.com/AmLtbwh.jpeg">
<figcaption>eCitadel cable soup.</figcaption>
</figure>

This year we're charging registration fees of $10 per team to participate in eCitadel. All registration fees will go towards supporting things like:

- Infrastructure (hosting, bandwith, etc)
- Time spent running eCitadel events
- Prizes
- Merch (stickers, coins)

### Competition Window

The competition window will be increasing from 4 to 6 hours. We feel that this will allow teams to become more familiar with the environment and handle more challenges, and overall allow for a better competition experience.

### Web-based challenges

We're removing web-based challenges from eCitadel to focus on injects that relate more to the competition environment. CTF haters, rejoice! And for you die-hard CTF fans out there, don't worry - we have plans to host an eCitadel-style CTF competition in the future. We want to take the time to get things right, so you can expect more updates towards the end of the year.

### Service Scoring

We feel that scoring services externally worked out really well last year. This year we'll be scoring your services live during your 6-hour competition window (as opposed to just one service check). Critical services will account for roughly 20 percent of the possible points.

### Injects

The introduction of injects was well-received last year, and this year injects will account for roughly 40 percent of the possible points. We're going to be introducing different types of injects, including writing business-style reports that will be manually graded. Teams should expect to have to put a lot of effort into writing high-quality reports to achieve a high score.

### Red Team

Teams will have simulated red team activity on their machines, including pre-planted malware. Successful red team persistence will result in point deductions from a team's total score. Incident reports may be submitted and will be part of a team's inject scores.

### VPN

The competition will remain entirely browser-based, but we will give teams the option to connect to a VPN if they want to have direct SSH/RDP access to their machines.

### Vulnerabilities

The machines will have less scored vulnerabilities overall, but they will be more focused in specific categories.

## Summary

With these new changes, our goal is to make it realistic for a well-prepared team to achieve a perfect or near-perfect score. We're looking forward to hosting the competition and appreciate all the enthusiasm and support you all have shown so far. You can now sign up for the Season III of the eCitadel Open! Visit [https://ecitadel.org/registration](https://ecitadel.org/registration){:target="_blank"} to learn how to register.
