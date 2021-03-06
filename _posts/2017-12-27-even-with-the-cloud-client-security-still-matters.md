---
layout: post
title: "Even With the Cloud, Client Security Still Matters"
category: Security
tags:
  - Security
  - Cloud
excerpt:
  Despite the move of resources to the cloud, security of your clients and
  endpoints remains important.  Some believe that only servers need to be
  secured, but it's important to remember that the clients with access to
  servers are equally vulnerable to compromise and use to gain access.
date: 2017-12-27
---

**As usual, this post does not necessarily represent the views of my employer
(past, present, or future).**

It's Friday afternoon and the marketing manager receives an email with the new
printed material proofs for the trade show.  Double clicking the PDF attachment,
his PDF reader promptly crashes.

"Ugh, I'm gonna have to call IT again.  I'll do it Monday morning," he thinks,
and turns off his monitor before heading home for the weekend.

Meanwhile, in a dark room somewhere, a few lines appear on the screen of a
laptop:

```
[*] Sending stage (205891 bytes) to 10.66.60.101
[*] Meterpreter session 1 opened (10.66.60.100:4444 -> 10.66.60.101:49159) at 2017-12-27 16:29:13 -0800
msf exploit(multi/handler) > sessions 1
[*] Starting interaction with 1...
meterpreter > sysinfo
Computer        : INHUMAN-WIN7
OS              : Windows 7 (Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : ENTERPRISE
Logged On Users : 2
Meterpreter     : x64/windows
```

Finally, the hacker had a foothold.  He started exploring the machine remotely.
First, he used [Mimikatz](https://github.com/gentilkiwi/mimikatz) to dump the
password hashes from the local system.  He sent the hashes to his computer with
8 NVidia 1080Ti graphics cards to start cracking, and then kept exploring the
filesystem of the marketing manager's computer.  He grabbed the browsing history
and saved passwords from the browser, and noticed access to a company directory.
He started a script to download the entire contents through the meterpreter
session.  He started to move on to the network shares when his password cracking
rig flashed a new result.

"That was fast," he thought, looking over at the screen.  "SuperS3cr3t isn't
much of a password."  He used the password to log in to the company's webmail
and forwarded the "proofs" (in fact a PDF exploiting a known bug in the PDF
reader) to one of the IT staffers with a message asking them to take a look at
why it wouldn't render.

Dissatisfied with waiting until the next week for an IT staffer to open the
malicious PDF, he started looking for another option.  He began by using his
access to a single workstation to look for other computers that were vulnerable
to some of the most recent publicly known exploits.  Surprisingly, he found two
machines that were vulnerable to
[MS17-010](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue).
He sent the exploit through his exisiting meterpreter session and crossed his
fingers.

Moments later, he was rewarded with a second Meterpreter session.  Looking
around, he was quickly disappointed to realize this machine was freshly
installed and so would not contain sensitive information or be hosting
interesting applications.  However, after running Mimikatz again, he discovered
that another one of the IT staff had logged into this machine, probably as part
of the setup process.

He threw the hashes into his password cracking rig again and started looking for
anything else interesting.  In a few minutes, he realized this machine was
devoid of anything but a basic Windows setup -- not even productivity
applications had been installed yet.  He returned to the original host and
looked for anything good, but only found a bunch of marketing materials that
were basically public information.

Frustrated, he banged on his keyboard until he remembered the scraped company
directory.  He went and looked at the directory information for the IT staffer
and realized it not only included names and contact informatuon for employees,
but also allowed employees to include information about hobbies and interests,
plus birthdays and more.  He took the data from the IT staffer, split it up into
all the included words, and placed it into a wordlist for his password cracking
rig.  Hoping that would get him somewhere, he went for a Red Bull.

When he came back, he saw another result on his password cracker.  This
surprised him slightly, because he had expected more of an IT staffer.  He was
even more surprised when he saw that the password was "Snowboarding2020!"
Though it met all the company's password complexity requirements, it was still
an incredibly weak password by modern standards.

Using this new found password, he logged into the workstation belonging to the
IT engineer.  He dumped the local hashes to look for further pivoting
opportunities, but found only the engineer's own password hash.  As he started
exploring the filesystem, however, he found many more interesting options.  He
quickly located an SSH private key and several text files containing AWS API
keys.  It only took a little bit of investigation to realize that one of the AWS
API keys was a root API key for the company's production environment.

Using the API key, he logged in to the AWS account and quickly identified the
virtual machines running the company's database servers containing user
credentials and information.  He connected with the API keys he had and started
dumping the usernames and password hashes.  Given that the hashes were unsalted
SHA-1, he figured it shouldn't take long for his password cracking rig to work
through them.

A day later, he was posting an offering for the plaintext credential database
for just a fraction of a bitcoin per customer.  Satisfied, he started hunting
for the next vulnerable enterprise.

* * *

While the preceeding story was fiction, it's an all too-common reality.  Many
modern enterprises have put considerable effort into hardening their datacenter
(be it virtualized or physical) but very little effort into hardening
workstations.  I often work with companies that seem to believe placing their
applications into the cloud is a security panacea.  While the cloud offers
numerous security benefits -- major cloud providers have invested heavily into
security, monitor their networks 24/7, and a cloud service is clearly heavily
segregated from the corporate network -- it does not solve all security
problems.

An attacker who is able to compromise a workstation is able to do anything that
a legitimate user of that workstation would be able to do.  In the example
above, the AWS keys stored on a workstation proved critical to gaining access to
a treasure trove of user information, but even a lower level of access can be
useful to an attacker and dangerous to your company.

The [2017 Verizon DBIR](http://www.verizonenterprise.com/verizon-insights-lab/dbir/2017/)
provides data to support this.  66% of malware began with malicious email
attachments (client-based), 81% of breaches involved stolen credentials
(pivoting), and 43% of attacks involved social engineering (tactics against
legitimate users).

Imagine the you have customer service representatives who log in to an
application hosted in the cloud to process refunds or perform other services.
An attacker with access to a customer service workstation might be able to grab
their username and password (or saved cookies from the browser) and then use it
to buy expensive items and refund them to themselves.  (Or change delivery
addresses, issue store credits, or other costly expenditures.)

In a hospital, compromising a workstation used by doctors and nurses would lead,
at a minimum, to a major HIPAA breach.  In the worst case, it could be used to
modify patient records or order medications what could be dangerous or fatal to
a patient.  Each environment needs to consider the risks posed by the access
granted from their workstations and clients.

Attackers will take the easiest route to the data they seek.  If you've spent
some effort on hardening your servers (or applications in the cloud), that may
well be through the workstation or client.  Consider all entry points in your
security strategy.
