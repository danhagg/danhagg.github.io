---
title: iPhone Third-party Apps
slug: third-party-apps
category: iPhone
tags: iPhone, iOS, Artifacts
Authors: Daniel Haggerty
---

# Where might we find third-party app data?

Look in info.plist  > Installed Applications > BundleId e.g., **com.kik.chat**

Data may be in either **AppDomain** or **AppDomainGroup**. 

For example, **AppDomain-com.kik.chat**
- Each of these folders hould contain the subfolders of Library/Preferences and a **plist** such as **com.kik.chat.plist** or **group.com.kik.chat.plist**

This **plist** may contain a ton of data. It is up to the third-party developer as to what information is stored in this **plist**

Can use a third-party app to find Buble Ids such as [OffCornerDev](https://offcornerdev.com/bundleid.html)


