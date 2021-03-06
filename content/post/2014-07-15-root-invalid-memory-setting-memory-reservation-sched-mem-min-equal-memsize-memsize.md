---
author: Christian Mohn
comments: true
date: 2014-07-15 10:59:42+00:00
layout: post
slug: root-invalid-memory-setting-memory-reservation-sched-mem-min-equal-memsize-memsize
title: 'Root Cause of Invalid memory setting: memory reservation (sched.mem.min) should
  be equal to memsize (memsize)'
url: /vmware-2/root-invalid-memory-setting-memory-reservation-sched-mem-min-equal-memsize-memsize/
wordpress_id: 3270
categories:
- VMware
tags:
- '5.5'
- ESXi
- Latency Sensitivity
- Troubleshooting
- vCenter
- VM
- VMware
---

[![Screenshot 2014-07-12 20.04.31](/img/Screenshot-2014-07-12-20.04.31-300x168.png)](/img/Screenshot-2014-07-12-20.04.31.png)While working on reconfiguring my home lab setup, and migrating all the vSphere resources into a single cluster I ran into a problem powering on one of the VMs which used to run on a single host. The power on operation yielded the following error message:

**_Invalid memory setting: memory reservation (sched.mem.min) should be equal to memsize (memsize)_**

<!--more-->

<blockquote>
<table border="0" >
<tbody >
<tr >

> <td colspan="2" >**Task Details:**
> </td>
</tr>
<tr >

> <td >
> </td>

> <td >
<table border="0" >
<tbody >
<tr >

> <td valign="top" >Status:
> </td>

> <td >Invalid memory setting: memory reservation (sched.mem.min) should be equal to memsize(4096).
> </td>
</tr>
<tr >

> <td valign="top" >Start Time:
> </td>

> <td >Jul 12, 2014 4:22:21 PM
> </td>
</tr>
<tr >

> <td valign="top" >Completed Time:
> </td>

> <td >Jul 12, 2014 4:22:23 PM
> </td>
</tr>
<tr >

> <td valign="top" >State:
> </td>

> <td >error
> </td>
</tr>
</tbody>
</table>

> </td>
</tr>
<tr >

> <td colspan="2" >**Error Stack:**
> </td>
</tr>
<tr >

> <td >
> </td>

> <td >

>
> ![](https://192.168.5.12:9443/vsphere-client/errorReport/assets/errorStack.png) An error was received from the ESX host while powering on VM MinecraftServer.
>
>

>
> ![](https://192.168.5.12:9443/vsphere-client/errorReport/assets/errorStack.png) Failed to start the virtual machine.
>
>

>
> ![](https://192.168.5.12:9443/vsphere-client/errorReport/assets/errorStack.png) Module MemSched power on failed.
>
>

>
> ![](https://192.168.5.12:9443/vsphere-client/errorReport/assets/errorStack.png) An error occurred while parsing scheduler-specific configuration parameters.
>
>

>
> ![](https://192.168.5.12:9443/vsphere-client/errorReport/assets/errorStack.png) Invalid memory setting: memory reservation (sched.mem.min) should be equal to memsize(4096).
>
> </td>
</tr>
<tr >

> <td colspan="2" >**Additional Task Details:**
> </td>
</tr>
<tr >

> <td >
> </td>

> <td >
<table border="0" >
<tbody >
<tr >

> <td valign="top" >VC Build:
> </td>

> <td >1476327
> </td>
</tr>
<tr >

> <td valign="top" >Error Type:
> </td>

> <td >GenericVmConfigFault
> </td>
</tr>
<tr >

> <td valign="top" >Task Id:
> </td>

> <td >Task
> </td>
</tr>
<tr >

> <td valign="top" >Cancelable:
> </td>

> <td >false
> </td>
</tr>
<tr >

> <td valign="top" >Canceled:
> </td>

> <td >false
> </td>
</tr>
<tr >

> <td valign="top" >Description Id:
> </td>

> <td >Datacenter.ExecuteVmPowerOnLRO
> </td>
</tr>
<tr >

> <td valign="top" >Event Chain Id:
> </td>

> <td >20017
> </td>
</tr>
</tbody>
</table>

> </td>
</tr>
</tbody>
</table>
</blockquote>



Clearly there was an issue with memory reservations on the VM, but there was no memory reservation enabled on it at all, nor should there be. The only related errors I found while investigating the issue, was with regards to [pass-through devices,](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2002779) which also did not apply in this case. It turns out that the problem was due to the VM was configured to use the latency sensitivity feature introduced in vSphere 5.5.

The [Deploying Extremely Latency-Sensitive Applications in VMware vSphere 5.5](http://www.vmware.com/files/pdf/techpaper/latency-sensitive-perf-vsphere55.pdf) whitepaper from VMware clearly states that usage of this feature also demands a memory reservation being set on the VM, and this VM had no reservation.



<blockquote>Enabling the latency-sensitivity feature requires memory reservation.</blockquote>



**In the end, the solution was a simple one, revert the latency [![Screenshot 2014-07-12 21.10.12](/img/Screenshot-2014-07-12-21.10.12-300x168.png)](/img/Screenshot-2014-07-12-21.10.12.png)sensitivity advanced option for the VM to the default value of _Normal_ let me power-on the VM again without issues.**

The error message received in vCenter could be a lot clearer though, and a knowledge base article with the exact error message and resolution paths might be in order. It was not immediately obvious that the lack of memory reservation error message was related to the latency sensitivity settings for the given VM. The vSphere Web Client shows a warning that you should check the CPU reservations, but does not mention memory reservations when you enable this feature.

Now I just need to figure out why my home MineCraft server had that setting enabled in the first place...
