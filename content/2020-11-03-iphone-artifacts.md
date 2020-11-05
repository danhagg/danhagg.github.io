---
title: iPhone Artifacts
slug: iphone-messaging-and-calls
category: iPhone
tags: iPhone, iOS, Artifacts
Authors: Daniel Haggerty
---

1. Acquisition
2. Apple Device Identifiers
3. iTunes Backups, info.plist
4. iMessage and SMS/MMS Messages
5. Call logs
6. Contacts
7. Calendars and Reminders
8. Safari
9. Third-party app data
10. Anatomy of an Application

## 1. Acquisition

- Property lockdown pairing plist will be located at C:\ProgramData\Apple\Lockdown
- Can copy **plist** from target comp to examiner. 
- May unlock if pairing cert is valid
- Jailbroken devices will reveal all of logical data tho may be encrypted

## 2. Apple Device Identifiers

Apple devices have several identifying values
- IMEI (international mobile equipment identity)
- MEID (mobile equipment identifier)
- Serial number assigned by Apple
- Unique Device Identifier or UDID (uniquely ID each device Apple).

### UDID
- The UDID is a 40-character alphanumeric value.
- Hashed unique values on device: chip ID, Wi-Fi/Bluetooth MAC
- UDID = SHA1(serial + IMEI or ECID + WiFiMac + BluetoothMac)
- The UDID is used by Apple to track devices. 
- Uniquely identify a backup on PC. 
    - If iOS plugged: UDID stored in Windows registry
- There are two ways to find out a devices UDID:
    1. Unlocked/paired to PC: When iOS device connected iTunes can show UDID: device properties and clicking on Serial No. == UDID
    2. Run **devmgmt.msc** (with iphone plugged in)
        - Expand USB devices to find Apple
        - Details > Property submenu > Device Instance Path
        - Value after \
- **Apple** or **05AC** 

## 3. iTunes Backups
- iOS backups on a PC almost identical to data on device
- iTunes tend to backup to PC's at following location:
**C:\Users<username>\AppData\Roaming\Apple Computer\MobileSync\Backup**
- May be possible to find multiple copies of same device under separate users.
- \Backup\ folder, subfolder folders for each backed-up device/UDID. 
- A UDID may be listed multiple times: appended by a 24hr date/time.
- Files will have no file extension & represent a file on the iOS device. 
- Filenames reflect a SHA1 value of the path of original device file. 
- Apple does not store full path, uses a list of Domains (shortcuts) eg:
    - AppDomain-com.some.user.installed.app
- Apple calculates hash using domain followed by the rest of the path. 
- eg, **/private/var/mobile/Library/SMS/sms.db** is represented in the backup as **HomeDomain-Library/SMS/sms.db** whose hash is **3d0d7e5fb2ce288813306e4d4636395e047a3d28**
- This hash reflect the SQLite database containing these messages.
- It will be a common hash for many devices
- A list of all paths & domains can be found in the **manifest.mbdb** file.
- iOS 10, Apple changed from **.mbdb** file to SQLite db **Manifest.db**.
    - **Manifest.db** contains two tables (Files and Properties)
    - **Files** table contains the following fields
        - **fileID** SHA1 hash for the object
        - **domain** domain the object belongs to
        - **relativePath** path from domain to object
        - **flags** is encrypted, is file, is folder
        - **file** binary plistfile stored as binary large object (BLOB)
- Along with organizational files and SHA1-files there are three property list files 
    - Manifest.plist 
    - Status.plist 
    - Info.plist 
- These files do not belong on device itself but contain info about device
    1. **Info.plist** is an XML file:
        - Static header 8-bytes of **bplist00**
        - Device Name
        - Display Name
        - GUID
        - ICCID (Integrated Circuit Card Identifier)
        - IMEI
        - Last Backup Date
        - Phone Number
        - Product Type
        - Product Version (iOS Version)
        - Serial Number
        - UDID
        - iTunes Version
        - List of Installed Applications
        - List of Applications in Library
    2. **Manifest.plist** is a binary file: 
        - whether or not the backup is encrypted
        - UDID of the device
        - whether or not a passcode was set on the device
        - Application data
        - Applications key including bundle ID, bundle version
        - **WasPasswordSet** 
        - **IsEnrypted** (was backup encryption enabled)
    3. **Status.plist** file
        - UDID
        - the state of the backup
        - whether or not it is a full backup.

### Domains
- All the domains are located in **Domains.plist** stored in **/system/library/backup**.
- This file is not backed up

### Encrypted backups
- \Backup\s select backup and SAVE FILE TO case folder
- Close Examine, Open Process to add backup/integrate
- Browse to current case, SCAN2, GO TO EVIDENCE SOURCES
- iOS>LOAD > FILES/FOLDERS > BACKUP > ENTER ENCRYPT PASSWORD

## 4. iMessage and SMS/MMS Messages

### Messages stored on device 

- All three message types (SMS, MMS, iMessage) stored in a SQLite3 database **sms.db**
- Located at **/private/var/mobile/Library/SMS/** or **HomeDomain-Library/SMS**
- **mobile** is the root user on each iPhone
- Media attachemnets stored in the **MediaDomain-Library/SMS/Attachments**
- Seven tables in **sms.db** of which 3 are joins and 4 contain details (Message, Chat, Handle, Attachment)
- Database stores times as Mac Epochs (1st, January, 2001 in seconds)
- Date, Date read, Date Received

### Sending and receiving messages

- iMessages can be transmitted over WiFi and/or cellular data
- iMessages can only be sent to other iMessage users (sent message = blue)
- If the message sent via SMS message the sent message will be green
- Dates/times
- Can determine those messages by those sent by Siri but *NOT* those sent by talk-to-text

More info at [Sanderson Forensics](https://sqliteforensictoolkit.com/sms-recovered-records-and-contacts-3-ways/)

## 5. Call logs

- Pre iOS 8 call logs stored in a SQLite3 database **call_history.db** in **WirelessDomain** located at **/private/var/wrieless**
- Modern call logs database is **CallHistroy.storedata** in **HomeDomain** located at **/Library/CallHistoryDB**
- **zservice_provider** tells you the application making the call
    - com.apple.Telephony is the standard iPhone call
    - com.apple.FaceTime
    - WhatsApp and Skype appear here if used
- **zcalltype** (1 standard, 8 FaceTime video, 16 FaceTime aduio)
- **zdate** is in Apple time
- If you remove application the call logs tend to disappear
- Application calls stored often exist only for a few days (e.g., Snapchat)

## 6. Contacts

- Stored in **AddressBook.sqlitedb** in **WirelessDomain** located at **/private/var/wrieless**
- Two tables of note
    1. **ABPerson** (Name, Organization, D.O.B)
    2. **ABMultiValue** (phone numbers, email addresses, ringtone settings)

## 7. Calendar & Reminders

- The calendar database is named  **calendar.sqlitedb** and is stored at **HomeDomain/Library/Calendar**
- The most important tables here are **Calendar** and **CalendarItem**
- **Calendar** stores calendars used by applications (iOS, Gmail, Facebook)
- **CalendarItem** stores information related to each entry saved
- **Reminders** are included in the **CalendarItem** table and linked to **Reminders** entry in **Calendar** table.

## 8. Safari

- Data storage for Safari has change dover the years and how data is stored will depend on version of iOS. 
- There has been a gradual shift from **plist** to **sqlite** databases.
- Safari's main history data is stored in a database named **History.db** 
- Located at **AppDomain-com.apple.mobilesafari/Library/Safari**
- Tables of interest are:
    - history_items (url, domain, visit count)
    - history_visits (visit time, page title)
- Artifacts extracted from Safari database may result from WebKit data: e.g., from internet searches from within applications.

## 9. Third-party app data

- Look in info.plist  > Installed Applications > BundleId e.g., **com.kik.chat**
- Data may be in either **AppDomain** or **AppDomainGroup**. 
- For example, **AppDomain-com.kik.chat**
    - Each of these folders hould contain the subfolders of Library/Preferences and a **plist** such as **com.kik.chat.plist** or **group.com.kik.chat.plist**
- This **plist** may contain a ton of data. It is up to the third-party developer as to what information is stored in this **plist**
- Can use a third-party app to find Bundle Ids such as [OffCornerDev](https://offcornerdev.com/bundleid.html)

- Majority of data on iPhones comes from the directory **/private/var/mobile**
- **/Library/** which holds most of the core user data; 
- **/Media/** camera roll, pictures, and videos on the device;
- **/Containers/** third-party and core application data on the device.
- Both types of property list files will store text data and Base-64 encoded data. 
- They may use the base-64 data to store other property list files, pictures, videos, or just additional text stores. 
- Binary and XML plist files: 
    - store user and configuration info. eg previous connections to wifi networks
- Also used in 3rd-party app data to store configuration info for each app (eg usernames, install date/times, last opened date/times etc)
- iOS uses plist files as registry files Contacts, Call Logs, SMS/iMessages
- Emails, Web History, and more are stored within SQLite databases.
1. plist xml header/footer:
    <?xml version=”1.0” encoding=”UTF-8”?>
    <!DOCTYPE plist SYSTEM “file://localhost/System/Library/DTDs/ PropertyList.dtd”>
    <plist version=”1.0>.
    </plist>
2. **plist** binary header **bplist00** 
3. SQLite3 header **SQLite format3** in ASCII
    - iOS uses SQLite version 3.7 which means they use write-ahead logs

## 10. Anatomy of an application
- Third-party **plist** files may contyain plain-text passwords

### Binary plist carving
- Export the plist
- Eaxamine in HxD
- [plist carver](https://github.com/pug4N6/plist_carver)