---
title: Disable Sending Emails Using Iptables
date: 2018-10-28 22:08:55
tags:
---


    #locate qmail 
    /usr/share/logwatch/scripts/services/qmail 
    /usr/share/logwatch/scripts/services/qmail-pop3d 
    /usr/share/logwatch/scripts/services/qmail-pop3ds 
    /usr/share/logwatch/scripts/services/qmail-send 
    /usr/share/logwatch/scripts/services/qmail-smtpd 
    
    iptables -A OUTPUT -p tcp –-dport 25 -j REJECT 

    /sbin/service iptables save 
