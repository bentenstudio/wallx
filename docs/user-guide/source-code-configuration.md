## Parse
 1. Open Config file and res/values/strings.xml
 2. Edit following keys with your Application ID and Client Key you created earlier on Parse Server

_Config.java_
```java hl_lines="1"
public static final String PARSE_APPLICATION_ID = "ENTER_YOUR_APPLICATION_ID";
public static final String PARSE_CLIENT_KEY = "ENTER_YOUR_CLIENT_KEY";
```

_res/values/strings.xml_
```
<string name="parse_app_id">ENTER_YOUR_APPLICATION_ID</string>
<string name="parse_client_key">ENTER_YOUR_CLIENT_KEY</string>
```
<br>
## Social Login
#### _Configure Facebook Login_
By default, the app will have Facebook login. You need to change following field in res/values/strings.xml so that this could work.
```
<string name="facebook_app_id">ENTER_YOUR_FACEBOOK_APP_ID</string>
```
<br>
#### _Disable Facebook Login_
If you wish to disable Facebook Login, you can remove following meta-data from AndroidManifest.xml file
```xml
<meta-data
 android:name="com.parse.ui.ParseLoginActivity.FACEBOOK_LOGIN_ENABLED"
    android:value="true" />
<meta-data
 android:name="com.parse.ui.ParseLoginActivity.FACEBOOK_LOGIN_PERMISSIONS"
    android:resource="@array/my_facebook_permissions" />
```
<br>
## App Intro
#### _Change slides content_
Open **res/values/strings.xml** and edit following fields
```xml
<string name="intro.slide1.title">WallX</string>
<string name="intro.slide1.description">Your ultimate wallpaper application</string>
<string name="intro.slide2.title">Easy Customization</string>
<string name="intro.slide2.description">This app Intro is easy to customize. So do other parts of the source.</string>
<string name="intro.slide3.title">Sold on Envato</string>
<string name="intro.slide3.description">This item is sold exclusively on Codecanyon</string>
```
<br>
#### _Change slides' images_
Replace following files in res/drawable with desired ones

 - intro_slide1.jpg 
 - intro_slide2.jpg 
 - intro_slide3.jpg

<br>
## About
#### _Change Social identities_
Edit following settings in Config.java
```java
public static final String FACEBOOK_PAGE = "bentenstudio";
public static final String TWITTER_ID = "bentenstudio";
public static final String EMAIL_ID = "support@bentenstudio.co";
```
<br>
#### _Change the background_
Replace **drawable/about_background.jpg** with your desired image