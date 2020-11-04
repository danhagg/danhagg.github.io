---
title: iPhone Messages, Calls and Contacts
slug: iphone-messaging-and-calls
category: iPhone
tags: iPhone, iOS, Artifacts
Authors: Daniel Haggerty
---

# iMessage and SMS/MMS Messages

## Messages stored on device 
- All three message types (SMS, MMS, iMessage) stored in a SQLite3 database **sms.db**
- Located at **/private/var/mobile/Library/SMS/** or **HomeDomain-Library/SMS**
- **mobile** is the root user on each iPhone
- Media attachemnets stored in the **MediaDomain-Library/SMS/Attachments**
- Seven tables in **sms.db** of which 3 are joins and 4 contain details (Message, Chat, Handle, Attachment)
- Database stores times as Mac Epochs (1st, January, 2001 in seconds)
- Date, Date read, Date Received

## Sending and receiving messages
- iMessages can be transmitted over WiFi and/or cellular data
- iMessages can only be sent to other iMessage users (sent message = blue)
- If the message sent via SMS message the sent message will be green
- Dates/times
- Can determine those messages by those sent by Siri but *NOT* those sent by talk-to-text

More info at [Sanderson Forensics](https://sqliteforensictoolkit.com/sms-recovered-records-and-contacts-3-ways/)

# Call logs
- Pre iOS 8 call logs stored in a SQLite3 database **call_history.db** in **WirelessDomain** located at **/private/var/wrieless**
- Modern call logs database is **CallHistroy.storedata** in **HomeDomain** located at **/Library/CallHistoryDB**
- zservice_provider tells you the application making the call
    - com.apple.Telephony is the standard iPhone call
    - com.apple.FaceTime
    - WhatsApp and Skype appear here if used
- zcalltype (1 standard, 8 FaceTime video, 16 FaceTime aduio)
- zdate is in Apple time
- If you remove application the call logs tend to disappear
- Application calls stored often exist only for a few days (e.g., Snapchat)

# Contacts
- Stored in **AddressBook.sqlitedb** in **WirelessDomain** located at **/private/var/wrieless**
- Two tables of note
    1. ABPerson (Name, Organization, D.O.B)
    2. ABMultiValue (phone numbers, email addresses, ringtone settings)