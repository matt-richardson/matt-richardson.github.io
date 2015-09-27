---
layout: post
title: " Nerd post: Citrix remote access crash "
date: 2009-05-04 07:43:44 +0100
comments: true
published: true
categories: ["blog", "archives"]
tags: []
alias: ["/mattsblog/archive/2009/05/03/nerd-post-citrix-remote-access-crash.aspx"]
---
<!-- more -->

<p>Be warned – you should probably only continue if you’re having issues with Citrix crashing when attempting to launch the remote desktop client.</p>
<p>Been fighting trying to connect to work via Citrix (Metaframe Presentation Server) for the last 6 weeks or so, and I’ve finally figured it out, so I’m posting this in the hope that it will save someone else a whole heap of time.</p>
<p>The issue was that I’d be able to login and get to the applications folder, but wasn’t able to do anything past that. It would download the ica file, then show up with either:</p>
<p>"The ordinal1191 could not be located in the dynamic link library MFC80.DLL":</p>
<figure>
    <img title="“The ordinal1191 could not be located in the dynamic link library MFC80.DLL”" border="0" alt="“The ordinal1191 could not be located in the dynamic link library MFC80.DLL”" src="/images/clip_image002_239FD24D.jpg" width="461" height="169">
</figure>
<p>or "The ordinal 2468 could not be located in the dynamic link library MFC80.DLL":</p>
<figure>
    <img title="The ordinal 2468 could not be located in the dynamic link library MFC80.DLL" border="0" alt="“The ordinal 2468 could not be located in the dynamic link library MFC80.DLL" src="/images/clip_image0027_26F14428.jpg" width="461" height="169">
</figure>
<p>followed by the standard windows error handling "Citrix ICA Client Engine (Win32) has stopped working":</p>
<figure>
  <img title="Citrix ICA Client Engine (Win32) has stopped working" border="0" alt="Citrix ICA Client Engine (Win32) has stopped working" src="/images/clip_image0024_35D87342.jpg" width="366" height="204">
</figure>
<p>The event log would come up with something like:</p>
<p style="font-family: courier;">Faulting application wfcrun32.exe, version 11.0.0.5357, time stamp 0x48a746f7, faulting module ntdll.dll, version 6.0.6002.18005, time stamp 0x49e03821, exception code 0xc0000138, fault offset 0x00009eed
</p>
<p>with a few more irrelevant bits.</p>
<p>After finding out far too much about WinSxs and VC++ 2005 SP1 redistributables, turns out there’s an issue that can potentially occur when you install Visual Studio 2008, or SQL Server 2008 (and potentially something to do with uninstalling Visual Studio 2005?). There’s a hotfix that you can get from Microsoft (KB961894) that resolves the issue: <a title="http://support.microsoft.com/kb/961894" href="http://support.microsoft.com/kb/961894">http://support.microsoft.com/kb/961894</a>.</p>
<p>Hope this helps someone!</p>
