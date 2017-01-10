---
layout: post
title: A date with postfix
---
To inspect the mail queue

    mailq
    postqueue -p

To inspect the message content in queue
    postcat -vq <message_id_from_mailq_or_postqueue_command>

To flus the queue/ forcing the postfix to process queue
    postqueue -f
    postfix flus
    

This is the reference link. http://www.tech-g.com/2012/07/15/inspecting-postfixs-email-queue/



**************************************

Was trying to send an email from mutt today from my vm box. But didn't receive it. On investigation, found that postfix was not running.
Even after trying to start it, postfix was not coming up.

I was getting this error in `systemctl status postfix` scan_dir_push: open directory defer: Permission denied

To fix it, I simply run `sudo postfix set-permissions` and then `systemctl start postfix`. Now potfix came up.
