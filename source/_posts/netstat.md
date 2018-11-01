---
title: netstat
date: 2018-10-28 22:14:43
tags:
---
	netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
	TIME_WAIT 1748
	CLOSE_WAIT 9
	FIN_WAIT1 11
	FIN_WAIT2 159
	ESTABLISHED 844
	SYN_RECV 17
	CLOSING 1
	LAST_ACK 37