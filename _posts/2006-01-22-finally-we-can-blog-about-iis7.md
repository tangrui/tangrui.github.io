---
ID: 15
title: FINALLY! We can blog about IIS7
author: 唐睿
date: 2006-01-22 22:32:00 +0800
categories: [技术]
tags: [IIS, Windows, Web]
layout: post
published: true
---

[Robert McLaws: FunWithCoding.NET - Longhorn Edition](http://www.longhornblogs.com/robert/default.aspx)

I saw IIS7 last year at the MVP Summit, and I've been really bummed because I haven't been able to talk about it. Today, the Group Program Manager, Bill Staples, finally allowed us to start talking about it.

IIS7 represents the unification of ASP.NET and IIS. Let me clarify what that means. Right now, ASP.NET is implemented as an ISAPI extension for IIS. That will still be true in ASP.NET 2.0. In IIS7, that changes. Instead, the concepts of HTTP pipelines, handlers, modules, XML config files, etc... are all natively built into the platform.

Along with that, the IIS7 team has completely refactored the whole platform, so now practically every feature in the pipeline has been broken out into a separate module. From a security standpoint, this is a whole new realm for IIS. Because the pipeline is componentized (the WHOLE pipeline), you can reduce your surface area by removing any module you're not using from the pipeline. So if you don't have Passport authentication, don't enable the module. IMO, that's really freakin cool.

My second-favorite feature is configuration. Microsoft is using the same configuration paradigm in ASP.NET for IIS7. That means that you won't have to configure options in two places, and you'll be able to delegate every configuration option to individual websites. You can use things like Forms Authentication to secure non-ASP.NET content, which should also be very helpful.

Finally, if IIS7 doesn't do something you want it to do, you can write an HttpHandler or HttpModule to support the functionality you need. Say you want to run PHP on IIS7... done. Say you want to be able to support serving up a new filetype... piece of cake. Forget about all the hassles of ISAPI... programming managed IIS extensions is far easier. Of course, ISAPI extensions will still work for compatibility reasons, but hopefully unmanaged extensions will be a thing of the past.

Why am I talking about all this on a Longhorn site? Well, right now Microsoft will be releasing IIS7 on Longhorn Server and Client only. That's as of right now. They haven't locked down whether or not it will ship downlevel, but they will decide on that soon. As a side note, the current bits are running on Windows Server 2003 SP1, so shipping downlevel is entirely possible.

And yes, IIS7 will be on Longhorn Client. They've taken out all the old site and connection limits, which is good news for many. The only limit that will be added to Longhorn Client is a request limit. After a certain number of concurrent requests are made, requests will be throttled back and processed through a request queue. But there will not be any site limits. Also, at this point, there will be a version of IIS7 in the "Home" SKUs of Longhorn. I must emphasize that MS's position on that may change.

I saved the most important info for last: ship schedule. Right now, the team is code complete on IIS7. They are currently working on integrating into Longhorn, which is targeted for the end of July. That means that IIS7 will not be a part of Longhorn until Beta 2.

I LOVE IIS7. I absolutely love it. I can't wait to put it into a production environment.
