**WallX: Material Wallpaper with Parse Server**
==========
---
**Created:** 18/09/2015
**Latest update:** 02/03/2016
**Author:** Benten Studio
**Email:** support@bentenstudio.co

---
##Table of Contents
[TOC]

---
## 1. Getting Started

Thank you for purchasing WallX! Before we continue with the installation and setup steps, you need following things:

 - A computer or laptop with Windows or Mac OS
 - Android Studio software installed with latest version (1.3.0+)
 - Android SDK installed with at least version 15
 - ~~Parse.com account to set up database~~ Own Parse Server

Once you have these requirements, this guide will lead you through following steps:

 1. Parse Server setup
 2. App Configuration & Customization
 3. Content Moderation
 4. Monetization
 5. Migration from Parse.com

However, this documentation will not cover following sections:

 - Android Studio installation
 - Android SDK installation


Instead, you can follow below links for the instructions:

 - https://developer.android.com/sdk/index.html
 - Android Studio guide by [Techotopia](http://www.techotopia.com/index.php/Setting_up_a_Windows%2C_Linux_or_Mac_OS_X_Android_Studio_Development_Environment)

---
## 2. Parse Server
There are many server providers out there, but DigitalOcean is recommended due to its cheap plans. You can get $10 credit (2-month worth) by registering [here](https://m.do.co/c/b32e35e3d2be)
#### 2.1. Setting Up VPS
**Step 1: Create a Droplet**

 1. Create a new Droplet [here](https://cloud.digitalocean.com/droplets/new)
 2. Choose Ubuntu 14.04
 3. Choose a size that meets your requirement
 4. Choose preferred datacenter region
 5. Check User Data box and enter following script
 6. Fill in other optional information and click Create
 7. Once done, you will receive server information in your email inbox

*User Data script:*

    #cloud-config
    apt_sources:
      # Enable MongoDB repository
      - source: deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.0 multiverse
        keyid: 7F0CEB10
        filename: mongodb.list
    apt_update: true
    packages:
      - mongodb-org
      - git
      - bc
      - nginx

**Step 2: Installing SSH Client**
You will need to install a SSH client to connect to the VPS. Some good clients to use:
  - [PuTTY](http://www.putty.org/)
  - [WinSCP](https://winscp.net)
  - [SuperPuTTY](https://github.com/jimradford/superputty/releases)
  - [PuTTYtray](https://puttytray.goeswhere.com/)
  - [KiTTY](http://www.9bis.net/kitty/)
Follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-putty-on-digitalocean-droplets-windows-users) to connect to DigitalOcean VPS using PuTTY.
#### 2.2. Installing Parse Server
**Step 1: Connecting to VPS using SSH channel** (See previous section for tutorial).

**Step 2: Install Node.js and Development Tools**
In PuTTY command window, type (or copy/paste) following commands into the terminal.

    cd ~
    curl -sL https://deb.nodesource.com/setup_5.x -o nodesource_setup.sh
    sudo -E bash ./nodesource_setup.sh
    sudo apt-get install -y nodejs build-essential git

**Step 3: Install Parse Server module**
Use `npm` to install the `parse-server` utility, the `pm2` process manager, and their dependencies, globally:

    sudo npm install -g parse-server pm2

Instead of running `parse-server` as **root** or your `sudo` user, we'll create a system user called **parse**:

	sudo useradd --create-home --system parse

Now set a password for **parse**:

	sudo passwd parse

You'll be prompted to enter a password twice.

Now, use the `su` command to become the **parse** user:

	sudo su parse

Change to **parse**'s home directory:

	cd ~


Enter the cloned repository then run installation script.

    cd ~/parse-server
    npm install

To start your parse-server, enter following command in `parse-server` directory. Check **Configuring Parse Server** below before starting.

    npm start

#### 2.3. Configuring Parse Server
In the same terminal, open `index.js` file using

    nano index.js

Following fields need to be changed:

 - `appId`
 - `masterKey`
 - `databaseURI`
 - `clientKey` (Add this field into `ParseServer` object)
 - `facebookAppIds` (Add this field as below)

Part of the file should be as followed:

```Javascript
	var api = new ParseServer({
	  databaseURI: databaseUri || 'mongodb://localhost:27017/dev',
	  cloud: process.env.CLOUD_CODE_MAIN || __dirname + '/cloud/main.js',
	  appId: process.env.APP_ID || 'MY_APP_ID',
	  masterKey: process.env.MASTER_KEY || 'MY_MASTER_KEY',
	  serverURL: process.env.SERVER_URL || 'http://api.myserver.com',
	  clientKey: 'MY_CLIENT_KEY',
	  facebookAppIds: 'MY_FACEBOOK_APP_ID'
	});
```
Now you can start the server.

#### 2.3. Storing files with AWS S3
Check tutorial here: https://github.com/ParsePlatform/parse-server/wiki/Storing-Files-in-AWS-S3

---
## 3. Source Code Configuration
#### 3.1. Parse
 1. Open Config file and res/values/strings.xml
 2. Edit following keys with your Application ID and Client Key you created earlier on Parse Server

**Config.java**
```java
public static final String PARSE_APPLICATION_ID = "ENTER_YOUR_APPLICATION_ID";
public static final String PARSE_CLIENT_KEY = "ENTER_YOUR_CLIENT_KEY";
```

**res/values/strings.xml**
```
<string name="parse_app_id">ENTER_YOUR_APPLICATION_ID</string>
<string name="parse_client_key">ENTER_YOUR_CLIENT_KEY</string>
```

#### 3.2. Social Login
##### _Facebook Login_
By default, the app will have Facebook login. You need to change following field in res/values/strings.xml so that this could work.
```
<string name="facebook_app_id">ENTER_YOUR_FACEBOOK_APP_ID</string>
```

##### _How to disable Facebook Login_
If you wish to disable Facebook Login, you can remove following meta-data from AndroidManifest.xml file
```xml
<meta-data
 android:name="com.parse.ui.ParseLoginActivity.FACEBOOK_LOGIN_ENABLED"
    android:value="true" />
<meta-data
 android:name="com.parse.ui.ParseLoginActivity.FACEBOOK_LOGIN_PERMISSIONS"
    android:resource="@array/my_facebook_permissions" />
```

#### 3.3. App Intro
##### _How to change slides content_
Open **res/values/strings.xml** and edit following fields
```xml
<string name="intro.slide1.title">WallX</string>
<string name="intro.slide1.description">Your ultimate wallpaper application</string>
<string name="intro.slide2.title">Easy Customization</string>
<string name="intro.slide2.description">This app Intro is easy to customize. So do other parts of the source.</string>
<string name="intro.slide3.title">Sold on Envato</string>
<string name="intro.slide3.description">This item is sold exclusively on Codecanyon</string>
```
##### _How to change slides' images_
Replace following files in res/drawable with desired ones

 - intro_slide1.jpg 
 - intro_slide2.jpg 
 - intro_slide3.jpg

#### 3.4. About
##### _How to change Social identities_
Edit following settings in Config.java
```java
public static final String FACEBOOK_PAGE = "bentenstudio";
public static final String TWITTER_ID = "bentenstudio";
public static final String EMAIL_ID = "support@bentenstudio.co";
```
##### _How to change the background_
Replace **drawable/about_background.jpg** with your desired image

---
## 4. Content Moderation
#### 4.1. Moderation Mode

Moderation mode means that all submissions will need to be approved before showing in app. 

To turn this on, open Config.java file and change **MODERATION_MODE** to **true**. This value is **false** by default.

#### 4.2. Photo Approval
By default, any photo that was uploaded by users will have this value as false. It is only true when photos are uploaded by Dashboard or it was marked as true later in the Dashboard >> Moderation

To approve a photo, login to Dashboard and browse to Moderation section. Pick the photo that needs to be moderated and click "Approve" button. 

Or you can manually change `isApproved` value to `true` in any wallpaper object.

#### 4.3. Flag Threshold
Flag threshold: a value which decides if a photo is appropriate or not. This value is increased whenever a logged-in user choose "Flag inappropriate" option in app. Any approved photo will be displayed no matter how high this value is.

---
## 5. Monetization
#### 5.1. Ads Toggle

The app support banners and interstitial ads which show up when users quit. Open Config file and edit following variable to enable or disable ads.

 - SHOULD_SHOW_ADS
 - SHOULD_SHOW_BANNERS
 - SHOULD_SHOW_INTERSTITIAL
The expected values should be **true** or **false**


#### 5.2. Ad Unit Setup

Open **res/values/strings.xml** and edit following fields:
```xml
<string name="banner_ad_unit_id">ca-app-pub-3940256099942544/6300978111</string>
<string name="interstitial_unit_id">ca-app-pub-3940256099942544/1033173712</string>
```

---
## 6. Migration from Parse.com
#### 6.1. Configuring MongoDB
If you have `Let's Encrypt` SSL certificates, run following command. Make sure to replace `myUsername` with yours.

    mongo --port 27017 --ssl --sslAllowInvalidCertificates --authenticationDatabase admin --username myUsername--password

You will be prompt to enter the password you created earlier.

Once logged in, you will need to create a database and a user that is used for migration. Run following command with replacing `database_name` with your own.

	use database_name
	db.createUser({ user: "parse", pwd: "password", roles: [ "readWrite", "dbAdmin" ] })

#### 6.2. Initiate Migration Process
Login to Parse dashboard at dashboard.parse.com then open the settings for your app. Under **General**, locate the **Migrate** button and click it:
![Migrate in General](%7B%7B%20site.url%20%7D%7D/assets/images/small-001.jpg)

You will be prompted for a MongoDB connection string. Use the following format:

	mongodb://parse:password@your_domain_name:27017/database_name

If you have `Let's Encrypt` SSL, add `?ssl=true` after the URL.

For example, if you are using the domain `example.com`, with the user `parse`, the password `foo`, and a database called `wallx`, your connection string would look like this:

	mongodb://parse:foo@example.com:27017/wallx?ssl=true

The configuration should look like below
![Database connection string](%7B%7B%20site.url%20%7D%7D/assets/images/small-002.jpg)

Click **Begin the migration**. You should see progress dialogs for copying a snapshot of your Parse hosted database to your server, and then for syncing new data since the snapshot was taken. The duration of this process will depend on the amount of data to be transferred, and may be substantial.
	
![](%7B%7B%20site.url%20%7D%7D/assets/images/small-003.jpg)
![](%7B%7B%20site.url%20%7D%7D/assets/images/small-004.jpg)

**Don't click Finalize yet.** We need to verify the migration first.

#### 6.3. Verify Data Migration
Return to your `mongo` shell, and examine your local database. Begin by accessing **database_name** and examining the collections it contains:

	use database_name
	show collections

Sample output

	Wallpaper
	Likes
	_Index
	_SCHEMA
	_Session
	_User
	_dummy
	system.indexes

You can examine the contents of a specific collection with the `.find() `method:

	db.Wallpaper.find()

---
## 7. Support
#### 7.1. Bug Fix
Send an email to support@bentenstudio.co including your purchase code and the issue.

#### 7.2. Other Questions
Feel free to contact us at support@bentenstudio.co