# KeePass OneDrive Sync

## Download ##

### Can I share one KeePass database with friends/co-workers/family without having to share my credentials to my OneDrive? ###

Yes! You can simply share the KeePass database or its entire folder on your OneDrive Personal or OneDrive for Business site. Be sure to share it with the people you want to share the KeePass database with by entering their e-mail addresses. Sharing the database through a shareable link will not work for this plugin.

Now simply have the other person do a File -> Open -> Open from OneDrive (ctrl+alt+o), have them log in to their own OneDrive Personal / OneDrive for Business site preferably using the Graph API, click on the "Shared with me" tab in the OneDrive file picker dialog and select the database that was shared. Note that it can take a few minutes between creating a new shared item and it actually showing up in the dialog. You can hit F5 to refresh the list to see when it shows up if it doesn't right-away.

Another option to share a KeePass database is to upload it to a SharePoint site on a SharePoint farm that supports ACS / Low Trust. You can then just share the Client ID / Client Secret which gives access to the database location to give others access to it as well.

### KeePass OneDrive Sync doesn't work through my proxy! ###

 See the Proxy Issues section in [Troubleshooting](./Troubleshooting.md)

### Can you make a version that works on non Windows machines? ###

No I cannot.

### Can you make a version that works with an older .NET Framework version than 4.5.2? ###

No I will not. Lots of functionality inside the plugin depends on the async development patterns introduced in the .NET 4.5 framework. I will not take the extreme efforts to use alternative approaches for this. Just install .NET 4.5 or later. If you can't because you're still on Windows XP: dude, upgrade your operating system!

### What happens to my KeePass database if multiple changes on multiple devices have been made? ###

When triggering a save (ctrl+s) or opening a KeePass database, the plugin will automatically verify against OneDrive Personal / OneDrive for Business if the file hosted there has been updated since the last time that specific client downloaded it. If not, it will not do anything. If so, it will download a copy of that KeePass database to a temporary location on your harddrive and merge all changes from the downloaded copy with your local KeePass database. So changes in both the version stored online as well as in your local copy will be retained. Not at any time will one overwrite the other. This is ideal in situations where you're using KeePass on clients that are not always online (i.e. laptops). As soon as you get online and trigger a sync, everything will be merged again. Once the changes are merged, an updated copy of the KeePass database will automatically be uploaded to OneDrive / OneDrive for Business again so your other devices can grab the updated copy.

### With Microsoft Graph API support having been added in v2.0, should I switch my current OneDrive Personal / OneDrive for Business syncs to use that instead? ###

Yes, you should. The OneDrive for Business option on the Other tab will stop working on November 5, 2019 or soon thereafter. The OneDrive option on the Other tab will remain working for the foreseeable future but will be deprecated by Microsoft at some point. The Microosft Graph options are the way forward. With the addition of the device ID login in version 2.1.0.0 there should be no reason anymore not to use the Microsoft Graph options.

### I want to switch from using the OneDrive API to using the Graph API, how do I do this? ###

Just go into the KeePass -> Tools -> OneDriveSync Options and delete the line(s) of the KeePass databases you wish to reconnect to a cloud storage provider. Once you open the database again and save it (CTRL+S), the wizard will pop up again allowing you to set up the synchronization. Just choose one of the two Microsoft Graph API options on the OneDrive tab and follow the steps.

### I reset my OneDrive password and now my KeePass sync fails, how do I fix this? ###

It is by design that when you reset your OneDrive (Microsoft Account) password, all active refresh tokens will be invalidated. This is a security measure as the reason for changing the password could be that somebody gained access to it. In this scenario your KeePass sync will stop working. You can easily resolve this by going Tools -> OneDriveSync Options -> delete the entry with the database you're having problems with. This will not delete the KeePass file, just the configuration for the plugin for it. Now if you save your KeePass database again (ctrl+s) you will receive the wizard again to set up your sync. After going through this again all should work well again.

### KeePass doesn't detect the plugin ###

If you have downloaded the PLGX and placed it inside the KeePass/Plugins folder (typically C:\Program Files (x86)\KeePass Password Safe 2\Plugins) and it doesn't show its functionality, ensure that the PLGX file is not blocked. By default it will be. Go to the Plugins folder, right click the KeeOneDriveSync.plgx file and go to its properties. If it shows an option to Unblock it at the bottom right of the General tab, check the box and hit OK. Restart KeePass. It should now properly load the plugin.

### Is there any (KeePass) data that flows through any of your environments? ###

No. There is no data that flows in any way to or through any service I host or own for this plugin. All communication goes directly between the KeePass client and the cloud provider where the data is hosted, such as Microsoft OneDrive for Business. The traffic between KeePass and Microsoft is encrypted through HTTPS encryption. The refresh token which could give access to the storage provider, such as OneDrive for Business, is stored to prevent having to authenticate over and over again on each synchronization. This token is stored either in the KeePass database, thus encrypted and secured in the same ways as everything else in your KeePass database is, or on your local file system in the user profile folder:

`C:\Users<username>\AppData\Roaming\KeePass`

The token in this config file is encrypted using built-in Windows encryption and only can be decrypted if you are logged on to Windows with the same user as under which this data is stored.

Communication with the storage providers happens via my [OneDriveAPI](https://github.com/KoenZomers/OneDriveAPI) open source API, as you can see in the [package reference](https://github.com/KoenZomers/KeePassOneDriveSync/blob/master/KoenZomers.KeePass.OneDriveSync/packages.config). If you want to see exactly where it specifies which services to communicate with, see here:

- [OneDrive for Business](https://github.com/KoenZomers/OneDriveAPI/blob/master/Api/OneDriveForBusinessO365Api.cs)
- [OneDrive Consumer](https://github.com/KoenZomers/OneDriveAPI/blob/master/Api/OneDriveConsumerApi.cs)
- [Microsoft Graph API](https://github.com/KoenZomers/OneDriveAPI/blob/master/Api/OneDriveGraphApi.cs)

You will find the URLs of the services it communicates with at the top of each file. You can see that these are all Microsoft owned and managed services and all communicate through HTTPS.

I recommend you to read up on the oAuth flow which will show you that all communication will always go between the client and the oAuth server directly, without having any third parties in between:

https://docs.microsoft.com/en-us/onedrive/developer/rest-api/getting-started/graph-oauth?view=odsp-graph-online

### How does the Microsoft Graph Any browser option work? ###

It utilizes the Microsoft Graph oAuth Device Code Flow. If you want the deep technical details on this, [read up here](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-device-code).

If you just want to understand the basic idea, it works as follows. When you choose this option when setting up the synchronization of your KeePass database with your OneDrive Consumer or OneDrive for Business site, the KeePass plugin will connect to the Microsoft Graph API to request a device login session. This will return a short unique identifier which will be shown to you by this plugin in your KeePass. You then open any internet browser you would like and navigate to the internet address shown in the KeePass dialog, which will typically be https://microsoft.com/devicelogin. You can even do this from any other device such as your tablet or phone. Enter the ID that is shown to you by the plugin in KeePass and go through the normal authentication process for your OneDrive Consumer or OneDrive for Business site. This process has full support for multi factor authentication and other identity providers you or your school or organization may have put in place such as AD FS, Ping Federate or one of the many others. Once authenticated, it may ask you to confirm granting the permission to access your files without you having to log on again to my plugin which will identify itself as "Koen Zomers OneDrive Sync v2". Once you grant it these rights, depending on how you have set up your account, it can be that you get a push notification on your phone, a text message on your phone and/or an e-mail stating that a new logon has just taken place under your account to the application "Koen Zomers OneDrive Sync v2". From here on the sync process works exactly like before.

### What shortcut keys can I use in the OneDriveSync dialog box ###

As of version 2.0.8.0 you can use these shortcut keys to quickly find your way in the OneDriveSync dialog box

  - F1: Opens the sync details screen. Only works when one KeePass database is selected.
  - F2: Allows renaming of the storage name for the KeePass database(s) you have selected
  - F4: Starts syncing the selected entries, if those databases are currently open in KeePass
  - F5: Refreshes the list with configuration entries
  - F7: Open the local file locations of the KeePass database(s) you have selected
  - F8: Open the selected KeePass database(s) in KeePass (new feature)
  - DEL: Remove the KeePass OneDriveSync configuration entries for the selected KeePass database(s). It will not remove the KeePass database itself, just the KeePass OneDriveSync configuration for it.
  - CTRL+A: Select all KeePass databases
  - CTRL+SHIFT+A: Select all KeePass databases that no longer exist locally (red colored background)
  - CTRL+Click: Select another KeePass database
  - SHIFT+Click: Select all KeePass databases between the currently selected one and the one you're clicking on
  - Use the right click menu to select all KeePass databases that haven't synced in either the last 24 hours, last week, last 2 weeks or last month

![](./Screenshots/KeePassOneDriveSyncConfigurationOptions.png)

### Other questions ###

Feel free to e-mail me at koen@zomers.eu or [open a GitHub Issue](https://github.com/KoenZomers/KeePassOneDriveSync/issues/new)
