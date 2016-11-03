<div class="library-github">** Contribute on GitHub **View Project | View File | Edit File</div>

This is a [Dazzle Software](https://dazzlesoftware.org/) Community guide for [Dazzle CMS](https://dazzlecms.org/). Write for us and earn $250 per published guide.

## Overview

There are many reasons why you would want to configure Postfix to send email using an external SMTP provider such as [Google Apps (Gmail)](#settings-for-google-apps), [Mandrill](#settings-for-mandrill), [SendGrid](#settings-for-sendgrid) or any other SMTP server. One reason is to avoid getting your mail flagged as spam if your current server’s IP has been added to a spam list. In this tutorial, you will learn how to install and configure a Postfix server to send email through [Google Apps](#settings-for-google-apps), [Mandrill](#settings-for-mandrill), or [SendGrid](#settings-for-sendgrid).

## What is Postfix?

Postfix is an MTA (Mail Transfer Agent), an application used to send and receive email. In this tutorial, we will install and configure Postfix so that it can be used to send emails by local applications only – that is, those installed on the same server that Postfix is installed on. Why would you want to do that? If you're already using a third-party email provider for sending and receiving emails, you, of course, do not need to run your own mail server. However, if you manage a cloud server on which you have installed applications that need to send email notifications, running a local, send-only SMTP server is a good alternative to using a 3rd party email service provider or running a full-blown SMTP server. An example of an application that sends email notifications is OSSEC, which will send email alerts to any configured email address (see [How To Install and Configure OSSEC Security Notifications on Ubuntu 16.04](https://codex.dazzlecms.org/tutorials/linux/install-configure-ossec-security-notifications-ubuntu-16-04)). Though OSSEC or any other application of its kind can use a third-party email provider's SMTP server to send email alerts, it can also use a local (send-only) SMTP server. That's what you'll learn how to do in this tutorial: how to install and configure Postfix as a send-only SMTP server.

> Note: If your use case is to receive notifications from your server at a single address, emails being marked as spam is not a significant issue, since you can whitelist them. If your use case is to send emails to potential site users, such as confirmation emails for message board sign-ups, you should definitely do [Protect Your Domain from Spammers](#protect-your-domain-from-spammers) section in this tutorial so your server's emails are more likely to be seen as legitimate. If you're still having problems with your server's emails being marked as spam, you will need to do further troubleshooting on your own.

### Prerequisites

Please complete the following prerequisites.

*   Ubuntu 16.04
*   Go through the [initial setup](https://codex.dazzlecms.org/tutorials/linux/initial-server-setup-with-ubuntu-16-04). That means you should have a standard user account with sudo privileges
*   Your fully qualified domain name (FQDN)
*   Your server's hostname should match this domain or subdomain. You can verify the server's hostname by typing hostname at the command prompt. The output should match the name you gave the DNS when it was being created, such as example.com
*   All updates installed :

<pre class="lang:sh decode:true">sudo apt-get update</pre>

*   A valid username and password for the SMTP mail provider, such as [Google Apps](#settings-for-google-apps), [MailChimp](#settings-for-mailchimp), [MailGun](#settings-for-mailgun), [Mandrill](#settings-for-mandrill), or [SendGrid](#settings-for-sendgrid)
*   Make sure the libsasl2-modules package is installed and up to date:

<pre class="lang:sh decode:true">sudo apt-get install libsasl2-modules</pre>

If all the prerequisites have been met, you're now ready for the [postfix installation](#postfix-installation) of this tutorial.

## Postfix Installation

In this step, you'll learn how to install Postfix. The most efficient way to install Postfix and other programs needed for testing email is to install the mailutils package by typing:

<pre class="lang:sh decode:true">sudo apt-get install mailutils
</pre>

Installing mailtuils will also cause Postfix to be installed, as well as a few other programs needed for Postfix to function. After typing that command, you will be presented with output that reads something like:

<pre class="lang:sh decode:true">The following NEW packages will be installed:
guile-2.0-libs libgsasl7 libkyotocabinet16 libltdl7 liblzo2-2 libmailutils4 libmysqlclient18 libntlm0 libunistring0 mailutils mailutils-common mysql-common postfix ssl-cert

0 upgraded, 14 newly installed, 0 to remove and 3 not upgraded.
Need to get 5,481 kB of archives.
After this operation, 26.9 MB of additional disk space will be used.
Do you want to continue? [Y/n]
</pre>

Press ENTER to install them. Near the end of the installation process, you will be presented with a window that looks exactly like the one in the image below. The default option is Internet Site. That's the recommended option for this tutorial, so press TAB, then ENTER. ![postfix-configuration](https://codex.dazzlecms.org/wp-content/uploads/2016/10/postfix-configuration.png) After that, you'll get another window just like the one in this next image. The System mail name should be the same as the name you assigned to the Domain when you were creating it. If it shows a subdomain like blog.example.com, change it to just example.com. When you're done, Press TAB, then ENTER. ![postfix-configuration-mail](https://codex.dazzlecms.org/wp-content/uploads/2016/10/postfix-configuration-mail.png) After installation has completed successfully, proceed to [configure postfix](#configure-postfix).

## Configure Postfix

In this step, you'll read how to configure Postfix to process requests to send emails only from the server on which it is running, that is, from localhost. For that to happen, Postfix needs to be configured to listen only on the loopback interface, the virtual network interface that the server uses to communicate internally. To make the change, open the main Postfix configuration file using the nano editor.

<pre class="lang:sh decode:true">sudo nano /etc/postfix/main.cf
</pre>

With the file open, scroll down until you see the entries shown in this code block.

<pre class="">mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
</pre>

Change the line that reads inet_interfaces = all to inet_interfaces = loopback-only. When you're done, that same section of the file should now read:

<pre class="">mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = loopback-only
</pre>

In place of loopback-only you may also use localhost, so that the modified section may also read:

<pre class="">mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = localhost
</pre>

When you're done editing the file, save and close it (press CTRL+X, followed by pressing Y, then ENTER). After that, restart Postfix by typing:

<pre class="lang:sh decode:true">sudo service postfix restart
</pre>

## Configuring SMTP Usernames and Passwords

Usernames and passwords are generally stored in a file called sasl_passwd in the `/etc/postfix/directory`. In this section, you’ll add your external mail provider credentials to this file and to Postfix. If you want to use [Google Apps](#settings-for-google-apps), [MailChimp](#settings-for-mailchimp), [MailGun](#settings-for-mailgun), [Mandrill](#settings-for-mandrill), or [SendGrid](#settings-for-sendgrid) as your SMTP provider, you may want to reference the appropriate example while working on this section. Open or create the `/etc/postfix/sasl_passwd` file, using your favorite text editor:

<pre class="lang:sh decode:true">sudo nano /etc/postfix/sasl_passwd</pre>

Add your destination (SMTP Host), username, and password in the following format: `/etc/postfix/sasl_passwd`

<pre class="lang:default decode:true">[mail.isp.example] username:password</pre>

If you want to specify a non-default TCP Port (such as 587), then use the following format: `/etc/postfix/sasl_passwd`

<pre class="lang:default decode:true">[mail.isp.example]:587 username:password</pre>

Create the hash db file for Postfix by running the postmap command:

<pre class="lang:sh decode:true">sudo postmap /etc/postfix/sasl_passwd</pre>

If all went well, you should have a new file named sasl_passwd.db in the `/etc/postfix/` directory.

## Securing Your Password and Hash Database Files

The `/etc/postfix/sasl_passwd` and the `/etc/postfix/sasl_passwd.db` files created in the previous steps contain your SMTP credentials in plain text. For security reasons, you should change their permissions so that only the root user can read or write to the file. Run the following commands to change the ownership to root and update the permissions for the two files:

<pre class="lang:sh decode:true">sudo chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db</pre>

## Configuring the Relay Server

In this section, you will configure the `/etc/postfix/main.cf` file to use the external SMTP server. Open the `/etc/postfix/main.cf` file with your favorite text editor:

<pre class="lang:sh decode:true">sudo nano /etc/postfix/main.cf</pre>

Update the relayhost parameter to show your external SMTP relay host. Important: If you specified a non-default TCP port in the sasl_passwd file, then you must use the same port when configuring the relayhost parameter. Open the `/etc/postfix/main.cf` file with your favorite text editor:

<pre class="lang:default decode:true"># specify SMTP relay host 
relayhost = [mail.isp.example]:587</pre>

> Check the appropriate [Google Apps](#settings-for-google-apps), [MailChimp](#settings-for-mailchimp), [MailGun](#settings-for-mailgun), [Mandrill](#settings-for-mandrill), or [SendGrid](#settings-for-sendgrid) section for the details to enter here.

At the end of the file, add the following parameters to enable authentication: Open the `/etc/postfix/main.cf` file with your favorite text editor:

<pre class="lang:default decode:true"># enable SASL authentication 
smtp_sasl_auth_enable = yes
# disallow methods that allow anonymous authentication. 
smtp_sasl_security_options = noanonymous
# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# Enable STARTTLS encryption 
smtp_use_tls = yes
# where to find CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt</pre>

Save your changes. Restart Postfix:

<pre class="">sudo service postfix restart</pre>

## Postfix Configurations with SMTP Providers

This section shows you settings for some popular mail services you can use as external SMTP servers. You may have to do some fine-tuning on your own to avoid Postfix logins being flagged as suspicious.

### Settings for Google Apps

Use these settings for [Google Apps](#settings-for-google-apps). Open the `/etc/postfix/sasl_passwd` with your favorite text editor, use the following configuration with your own credentials:

<pre class="lang:default decode:true">[smtp.gmail.com]:587 <USERNAME@gmail.com>:PASSWORD</pre>

If you are using [Google Apps](#settings-for-google-apps) with your own domain, configure `/etc/postfix/sasl_passwd` with:

<pre class="lang:default decode:true">[smtp.gmail.com]:587 <USERNAME@yourdomain.com>:PASSWORD</pre>

Open `/etc/postfix/main.cf` with your favorite text editor, use the following **relayhost**:

<pre class="lang:default decode:true">relayhost = [smtp.gmail.com]:587</pre>

Create the hash db file for Postfix by running the `postmap` command:

<pre class="lang:sh decode:true">sudo postmap /etc/postfix/sasl_passwd</pre>

Restart Postfix:

<pre class="lang:sh decode:true">sudo service postfix restart</pre>

### Settings for Mandrill

Use these settings for [Mandrill](#settings-for-mandrill). Open `/etc/postfix/sasl_passwd` with your favorite text editor, use the following configuration with your own credentials:

<pre class="lang:default decode:true">[smtp.mandrillapp.com]:587 USERNAME:API_KEY</pre>

Open `/etc/postfix/main.cf` with your favorite text editor, use the following **relayhost**:

<pre class="lang:default decode:true">relayhost = [smtp.mandrillapp.com]:587</pre>

Create the hash db file for Postfix by running the `postmap` command:

<pre class="lang:sh decode:true">sudo postmap /etc/postfix/sasl_passwd</pre>

Restart Postfix:

<pre class="lang:sh decode:true">sudo service postfix restart</pre>

### Settings for SendGrid

Use these settings for [SendGrid](#settings-for-sendgrid). Open `/etc/postfix/sasl_passwd` with your favorite text editor, use the following configuration with your own credentials:

<pre class="lang:default decode:true">[smtp.sendgrid.net]:587 USERNAME:PASSWORD</pre>

Open `/etc/postfix/main.cf` with your favorite text editor, use the following **relayhost**:

<pre class="lang:default decode:true">relayhost = [smtp.sendgrid.net]:587</pre>

Create the hash db file for Postfix by running the `postmap` command:

<pre class="lang:sh decode:true">sudo postmap /etc/postfix/sasl_passwd</pre>

Restart Postfix:

<pre class="lang:sh decode:true">sudo service postfix restart</pre>

## Test Postfix SMTP Server

In this step, you'll read how to test whether Postfix can send emails to any external email account. You'll be using the mail command, which is part of the mailutils package that was installed in Step 1. To send a test email, type:

<pre class="lang:sh decode:true">echo "This is the body of the email" | mail -s "This is the subject line" user@example.com</pre>

Alternatively, you can use Postfix’s own sendmail implementation, by entering lines similar to those shown below:

<pre class="lang:sh decode:true ">sendmail recipient@example.com
From: user@example.com
Subject: Test mail
This is a test email</pre>

In performing your own test(s), you may use the body and subject line text as-is, or change them to your liking. However, in place of [user@example.com](mailto:user@example.com), use a valid email address, where the domain part can begmail.com, fastmail.com, yahoo.com, or any other email service provider that you use. Now check the email address where you sent the test message. You should see the message in your inbox. If not, check your spam folder. Note: With this configuration, the address in the From field for the test emails you send will be [user@example.com](mailto:user@example.com), where user is your Linux username and the domain part is the server's hostname. If you change your username, the From address will also change.

## Forward System Mail

The last thing we want to set up is forwarding, so that you'll get emails sent to root on the system at your personal, external email address. To configure Postfix so that system-generated emails will be sent to your email address, you need to edit the `/etc/aliases` file.

<pre class="lang:sh decode:true">sudo nano /etc/aliases</pre>

The full content of the file on a default installation of Ubuntu 16.04 is shown in this code block:

<pre class="lang:default decode:true"># See man 5 aliases for format
postmaster: root</pre>

With that setting, system generated emails are sent to the root user. What you want to do is edit it so that those emails are rerouted to your email address. To accomplish that, edit the file so that it reads:

<pre class="lang:default decode:true"># See man 5 aliases for format
postmaster: root
root: user@example.com</pre>

Replace [user@example.com](mailto:user@example.com) with your personal email address. When done, save and close the file. For the change to take effect, run the following command:

<pre class="lang:sh decode:true">sudo newaliases</pre>

You may now test that it works by sending an email to the root account using:

<pre class="lang:sh decode:true">echo "This is the body of the email" | mail -s "This is the subject line" root</pre>

You should receive the email at your email address. If not, check your spam folder.

## Protect Your Domain from Spammers

In this step, you'll be given links to articles to help you protect your domain from being used for spamming. This is an optional but highly recommended step, because if configured correctly, this makes it difficult to send spam with an address that appears to originate from your domain. Doing these additional configuration steps will also make it more likely for common mail providers to see emails from your server as legitimate, rather than marking them as spam. [How To use an SPF Record to Prevent Spoofing & Improve E-mail Reliability](https://codex.dazzlecms.org/tutorials/linux/use-spf-record-prevent-spoofing-improve-e-mail-reliability) [How To Install and Configure DKIM with Postfix on Ubuntu](https://codex.dazzlecms.org/tutorials/linux/how-to-install-and-configure-dkim-with-postfix-on-ubuntu)

## Other Tutorials In This Series

Though the second article was written for Ubuntu 16.04, the same steps apply for any Debian or Ubuntu based distribution.