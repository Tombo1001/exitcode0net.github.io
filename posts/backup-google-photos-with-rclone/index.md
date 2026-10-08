# Backup Google Photos with Rclone

> Back up Google Photos to a local NAS or computer with Rclone. Automated offline copy of your entire photo library with WebDAV sync and scheduled cron jobs.

Source: https://exitcode0.net/posts/backup-google-photos-with-rclone/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-31
Updated: 2026-10-08
Tags: backups, go, google-photos, rclone



Google photos auto backup is a nice way to make sure that all your photos on your mobile phone are backed up. It takes the pain out of backups by allowing you to automatically upload your photos and videos to Google servers. For those who would also like an offline copy of those photos, it is now easy to automate this. Here is how to backup your Google Photos library to your local computer or NAS with Rclone.

[https://rclone.org/](https://rclone.org/ "https://rclone.org/")
Rclone is a command line program to sync files and directories to and from:

* Google Drive / Photos
* Dropbox
* Onedrive
* [and many more services](https://rclone.org/ "https://rclone.org/")

Rclone can actually copy data to these cloud services as well, but this particular article focuses on how to take your data back from the cloud – because I like to own my data.

Prerequisites
-------------

This is based on the assumption that:

* you are running Linux, mac OS or have a docker environment
* have Rclone installed – Install guide: [https://rclone.org/install/](https://rclone.org/install/ "https://rclone.org/install/")

Backup Google Photos with Rclone
--------------------------------

First, we must make a remote endpoint – Google Photos. So jump into Rclone config to do so:

```
rclone config
```

```
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> remote
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
[snip]
XX / Google Photos
   \ "google photos"
[snip]
Storage> google photos
** See help for google photos backend at: https://rclone.org/googlephotos/ **

Google Application Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id> 
Google Application Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret> 
Set to make the Google Photos backend read only.

If you choose read only then rclone will only request read only access
to your photos, otherwise rclone will request full access.
Enter a boolean value (true or false). Press Enter for the default ("false").
read_only> 
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> y
If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth
Log in and authorize rclone for access
Waiting for code...
Got code

*** IMPORTANT: All media items uploaded to Google Photos with rclone
*** are stored in full resolution at original quality.  These uploads
*** will count towards storage in your Google Account.

--------------------
[remote]
type: google photos
token: {"access_token":"XXX","token_type":"Bearer","refresh_token":"XXX","expiry":"2019-06-28T17:38:04.644930156+01:00"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
```

I was running this config script over a remote SSH session so actually chose not to use auto config. The only difference is the URL you are given sends you to a Google auth page to manually retrieve your session auth code.

Now that you have a remote endpoint setup, you can reference it in Rclone commands. This command is a simple way to take photos from my Google account and put them into Folders sorted by Year > Month:

```
rclone copy GooglePhotos:media/by-month /mnt/user/PhotosBackup
```

adding **–progress** to the end of that will give you a visual reference to the progress of the copy action.

Now what?
---------

From here you can set this command to run on a schedule using cron if you are using Linux:

```
crontab -e
```

A great resource for building cron tab commands: [http://www.cronmaker.com/](http://www.cronmaker.com/ "http://www.cronmaker.com/")



---
Markdown version of https://exitcode0.net/posts/backup-google-photos-with-rclone/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
