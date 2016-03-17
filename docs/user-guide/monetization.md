## Ads Toggle

The app support banners and interstitial ads which show up when users quit. Open `Config` file and edit following variable to enable or disable ads.

 - `SHOULD_SHOW_ADS`
 - `SHOULD_SHOW_BANNERS`
 - `SHOULD_SHOW_INTERSTITIAL`
 
The expected values should be **true** or **false**


## Ad Unit Setup

Open **res/values/strings.xml** and edit following fields:
```xml
<string name="banner_ad_unit_id">ca-app-pub-3940256099942544/6300978111</string>
<string name="interstitial_unit_id">ca-app-pub-3940256099942544/1033173712</string>
```