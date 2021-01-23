# Email server setup script

This is a heavily modified version of
[LukeSmith's emailwiz](https://github.com/LukeSmithxyz/emailwiz). Basically
everything (except comments) in the original script is rewrited to suit OpenBSD.

I wrote this script during the gruelling process of installing and setting up
an email server.  It perfectly reproduces my successful steps to ensure the
same setup time and time again.

Read this readme and peruse the script's comments before running it.  Expect it
to fail and you have to do bug testing and you will be very happy when it
actually works perfectly.

## This script installs

- **OpenSMTPD** to send and receive mail.
- **Dovecot** to get mail to your email client (mutt, Thunderbird, etc).
- Config files that unique the two above securely with native log-ins.
- **Rspamd** to prevent spam and allow you to make custom filters.
- **opensmtpd-filter-dkimsign** to validate you so you can send to Gmail and
  other big sites.

## This script does _not_

- use a SQL database or anything like that.
- set up a graphical interface for mail like Roundcube or Squirrel Mail. If you
  want that, you'll have to install it yourself. I just use
  [isync/msmtp/mutt-wizard](https://github.com/lukesmithxyz/mutt-wizard) to
  have an offline mirror of my email setup and I recommend the same. There are
  other ways of doing it though, like Thunderbird, etc.

## Requirements

1. A **OpenBSD server**. I've tested this on a
   [Vultr](https://www.vultr.com/?ref=8608122) OpenBSD server and their setup
   works, but I suspect other VPS hosts will have similar/possibly identical
   default settings which will let you run this on them. Note that the affiliate
   link there to Vultr gives you a $100 credit for the first month to play
   around.
2. **A Let's Encrypt SSL certificate for your site's `mta` subdomain**.
   Create a `httpd(1)` site at `mta.domain.tld` and get a certificate
   for it with `acme-client(1)`.
3. You need two little DNS records set on your domain registrar's site/DNS
   server: (1) an **MX record** pointing to your own main domain/IP and (2) a
   **CNAME record** for your `mta.` subdomain.
4. **A Reverse DNS entry for your site.** Go to your VPS settings and add an
   entry for your IPV4 Reverse DNS that goes from your IP address to
   `mta.domain.tld`. If you would like IPV6, you can do the same for
   that. This has been tested on Vultr, and all decent VPS hosts will have
   a section on their instance settings page to add a reverse DNS PTR entry.
   You can use the 'Test Email Server' or ':smtp' tool on
   [mxtoolbox](https://mxtoolbox.com/SuperTool.aspx) to test if you set up
   a reverse DNS correctly. This step is not required for everyone, but some
   big email services like gmail will stop emails coming from mail servers
   with no/invalid rDNS lookups. This means your email will fail to even
   make it to the receipients spam folder; it will never make it to them.
6. Some VPS providers block port 25 (used to send mail). You may need to
   request that this port be opened to send mail successfully. Although I have
   never had to do this on a Vultr VPS, others have had this issue so if you
   cannot send, contact your VPS provider.
7. Edit parameter section in emailwiz script. For example, change `${domain}` to
`changchukuan.name` and `${subdom}` to `mta`.

## Post-install requirement!

- After the script runs, you'll have to add additional DNS TXT records which
  are displayed at the end when the script is complete. They will help ensure
  your mail is validated and secure.
- Modify rspamd whitelists/blacklists in `/etc/rspamd/local.d` to yout need.

## Making new users/mail accounts

Let's say we want to add a user Billy and let him receive mail, run this:

```
useradd -m billy
passwd billy
```

A user's mail will appear in `~/Maildir/`. If you want to see your mail while
ssh'd in the server, you could just install mutt, add `set spoolfile="+Inbox"`
to your `~/.muttrc` and use mutt to view and reply to mail. You'll probably want
to log in remotely though:

## Logging in from Thunderbird or mutt (and others) remotely

Let's say you want to access your mail with Thunderbird or mutt or another
email program. For my domain, the server information will be as follows:

- SMTP server: `mta.domain.tld`
- SMTP port: 465
- IMAP server: `mta.domain.tld`
- IMAP port: 993
- Username `user` (I.e. *not* `user@domain.tld`)

The last point is important. Many email systems use a full email address on
login. Since we just simply use local BSDAuth logins, only the user's name is
used (this makes a difference if you're using luke's
[mutt-wizard](https://github.com/lukesmithxyz/mutt-wizard), etc.).

## Tweaking things

You're a big boy now if you have your own mail server!

You can tweak smtpd (sending mail

## Furthur reading
- [poolp's guide](https://poolp.org/posts/2019-09-14/setting-up-a-mail-server-with-opensmtpd-dovecot-and-rspamd/)

## Troubleshooting -- Can't send mail?

- Always check `/var/log/maillog` and `/var/log/rspamd` to see the specific
  problem.
- Go to [this site](https://appmaildev.com/en/dkim) to test your TXT records.
  If your DKIM, SPF or DMARC tests fail you probably copied in the TXT records
  incorrectly.
- If everything looks good and you *can* send mail, but it still goes to Gmail
  or another big provider's spam directory, your domain (especially if it's a
  new one) might be on a public spam list.  Check
  [this site](https://mxtoolbox.com/blacklists.aspx) to see if it is. Don't
  worry if you are: sometimes especially new domains are automatically assumed
  to be spam temporaily. If you are blacklisted by one of these, look into it
  and it will explain why and how to remove yourself.
