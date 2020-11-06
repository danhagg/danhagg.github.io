---
title: Android Artifacts
slug: android-artifacts
category: Android
tags: Android, Artifacts
Authors: Daniel Haggerty
---

Android phones do not immediately upgrades but rollouts tend to be taken up slowly (months to years).

Google is developer but manufacturers and carriers also have a role in upgrade process

Android devices now have full device encryption

### Android partitions include...
- **/boot**
- **/system**
- **/recovery**
- **/data** or **/userdata**
- **/cache**
- **/misc**
- **/sdcard**
- **/sd-ext**

Vendors may also specify device specific partitions. Not all partitions contain files or data.

- **/system**
    - Contains actual OS of Android other than kernel and ramdisk
    - Contains built-in system and vendor applications
- **/boot**
    - Contains kernel and ramdisk
- **/recovery**
    - Alternative boot partition. Allows for recovery and naintenance including wiping **/data**
    - Can be utilized to gain privileged access
- **/cache**
    - Application components/data
    - May store some system data for applications to access
    - Can be wiped without loss of functionality
- **/data**
    - Core user data
    - Third-party applications
    - Lots of juicy information in here for digital forensics

In the **/data** folder applications are sorted by Application ID.
Java package naming convention
- Companies use their reversed Internet domain name to begin their package names—for example, com.example.mypackage for a package named mypackage created by a programmer at example.com.

- **/userdata/data** is credential-encrypted storage
- **/user_de/0** is device encrypted storage
    - 0 is first user on device

## Contacts
- Stored at **/data/com.android.providers.contacts/databases/contacts2.db**
- Tables
    - **accounts**
        - Account name **myemail@gmail.com**
        - Account type **com.google**
    - **raw_contacts**
        - Display name for contact
        - Time of last interaction with said contact
    - **data**
        - Phone numbers
        - Addresses

## Call Logs
- Stored at **/data/com.android.providers.contacts/databases/contacts2.db**
- Tables
    - **accounts**
        - Account name **myemail@gmail.com**
        - Account type **com.google**
    - **raw_contacts**
        - Display name for contact
        - Time of last interaction with said contact
    - **data**
        - Phone numbers
        - Addresses
