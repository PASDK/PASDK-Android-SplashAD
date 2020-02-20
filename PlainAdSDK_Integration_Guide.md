# PlainAd SDK Integration

1. [Introduction](#introduction)

2. [Integration PlainAd SDK](#integration)

    [2.1 Integrating the PlainAd SDK in to project](#step1)

    [2.2 Initialize the PlainAd SDK](#step2)

    [2.3 GDPR](#step3)
    
    [2.4 Child Oriented Settings](#step4)

    [2.5 Android code obfuscation](#step5)
    
3. [Integration Notes](#note)

4. [Request Ad](#request)
    
    [4.1 Splash](#splash)
    
    
5. [Error Code For SDK](#error)

## <a name="introduction">Introduction</a>

- PlainAd SDK supports Banner, Interstitial, Native, Native Video andRewarded Video.
- PlainAd Android SDK supports Android API 15+.
- Please make sure you have added an app and at least one ad slot in PlainAd Platform.

## <a name="integration">2.Integration PlainAd SDK</a>  

### <a name="step1">2.1 Integrating the PlainAd SDK in to project</a>

* Check SDK in the rar.

* Detail of the different jars：

  | jar name                 | jar function                                    | require(Y/N) |
  | ------------------------ | ----------------------------------------------- | ------------ |
  | plainad_base_xx.jar        | basic functions(banner\interstitial\native ads) | Y            |
  | plainad_imageloader_xx.jar | imageloader functions                           | N            |
  | plainad_appwall_xx.jar     | appwall ads functions                           | N            |
  | plainad_video_xx.jar       | video ads functions                             | N            |

* Configure the module's build.gradle for basic functions：

``` groovy
    dependencies {
        compile files('libs/plainad_base_xx.jar')
    }
```

* Configure AndroidManifest.xml

```xml
	<!--Necessary Permissions-->
	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

	<!-- Necessary -->
	<activity android:name="com.xender.ad.splash.view.InnerWebViewActivity" />
	

	<!--If your targetSdkVersion is 28, you need update <application> as follows:-->
  	<application
    	...  	
        android:usesCleartextTraffic="true"
        ...>
        ...
    </application>
```



## <a name="step2">2.2 Initialization</a>   

**Set schema https**

```java
    PlainAdSDK.setSchema(true);
```

## <a name="step3">2.3 GDPR</a>  

**Use this interface to upload consent from affected users for GDPR**

```java
    /**
     * @param context       context
     * @param consentValue  whether the user agrees
     * @param consentType   the agreement name signed with users
     */
     PlainAdSDK.uploadConsent(this, true, "GDPR");
```
Warning:
1. If SDK don't gather the user informatian ,you probably get no fill. 
2. It is recommended to obtain the user's consent before the SDK request ad

## <a name="step4">2.4 Child Oriented Settings</a>  
In order to comply with the provisions of the Children's Online Privacy Protection Act (COPPA), we provide the setIsChildDirected interface.

Developers can use this interface to indicate that your content is child-oriented. We will stop personalized advertising and put in advertisements suitable for children，which may result in no filling.

``` java
     //child-oriented
     PlainAdSDK.setIsChildDirected(this, true);
```
Warning:
1. It is recommended to call this interface before requesting advertisements.

## <a name="step5">2.5 Obfuscation Configuration</a> 
> If it needs to obfuscate the codes in building the project process, you should add the following codes into the proguard file:

``` java
    #for sdk
    -keep public class com.xender.ad.splash.**{*;}
    -dontwarn com.xender.ad.splash.**

    #for gaid
    -keep class **.AdvertisingIdClient$** { *; }

    #for js and webview interface
    -keepclassmembers class * {
        @android.webkit.JavascriptInterface <methods>;
    }
    
```


## <a name="note">3.Integration Notes</a>

​	If you live in a country, such as China, which is forbidden google play, two prerequisites to get PlainAd ads: 
> * GooglePlay has installed on your device.
> * Your device have connect to VPN.

   

​	We suggest you define a class to implement the AdEventListener yourself , then you can just override the methods you need when you getBanner or getNative. See the following example:

``` java
public class MyPlainAdEventListener extends AdEventListener {

    /**
     * Get ad success
     */
    @Override
    public void onReceiveAdSucceed() {
    }

    /**
     * Get ad success
     * @param result Persistent object AdsVO
     */
    @Override
    public void onReceiveAdVoSucceed(AdsVO result) {
    }
    
    /**
     * Get ad failed Got data fail + Json fail + Rendering fail
     */
    @Override
    public void onReceiveAdFailed(String error) {
        Log.i("sdksample", "==error==" + error);
    }
    
    /**
     * Ad display success
     */
    @Override
    public void onShowSucceed(PANative result) {
    }

  
    /**
     * Go to landing page
     */
    @Override
    public void onLandPageShown(PANative result) {
    }

    /**
     * Ad was clicked
     */
    @Override
    public void onAdClicked(PANative result) {
    }

    /**
     * Ad is closed
     */
    @Override
    public void onAdClosed(PANative result) {
    }
}
```
## <a name="request">4.Request Ad</a>

## <a name="splash">4.1 Splash Ads Integration</a>

> Configure the AndroidManifest.xml for Splash AD

```xml
	<activity android:name="com.xender.ad.splash.view.SplashAdActivity" />    
```

> You need to add your own launch screen first. Then preload and show Splash AD.
>
> Preload Splash AD

``` java
    PlainAdSDK.preloadSplashAd(context, "Your Splash SlotID", new MyPlainAdEventListener() {
    
        @Override
        public void onReceiveAdSucceed() {
            Log.d(TAG, "Splash Ad Loaded.");
            show();//show splash ad
            finish();// close current activity
        }
    
        @Override
        public void onReceiveAdFailed(String error) {
            if (result != null)
                Log.e(TAG, "onReceiveAdFailed errorMsg=" + error);
        }     
    
    
    });
```

> Show ad from cache

```java
private void show() {

    PlainAdSDK.showSplashAd("Your Splash SlotID", new MyPlainAdEventListener() {
        @Override
	//impression, you can add customeView here showing your app name and icon (optional)
        public void onShowSucceed(PANative result) {
            if (result != null) {
                SplashView splashView = (SplashView) result;

                /*
                 * There are two ways to add a custom view
                 * inflate SplashView.getCustomParentView() or SplashView.addCustomView(view)
                 */
                 
                //1
                //LayoutInflater.from(getContext()).inflate(R.layout.custom_splash_layout, splashView.getCustomParentView(), true);

                //2
                LinearLayout linearLayout = new LinearLayout(result.getContext());
                linearLayout.setGravity(Gravity.CENTER);
                linearLayout.setBackgroundColor(Color.WHITE);
                linearLayout.setLayoutParams(new ViewGroup.LayoutParams(MATCH_PARENT, Utils.dpToPx(100)));
                TextView textView = new TextView(result.getContext());
                textView.setText("custom");
                textView.setTextSize(22);
                linearLayout.addView(textView, new ViewGroup.LayoutParams(WRAP_CONTENT, WRAP_CONTENT));
                linearLayout.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Toast.makeText(getBaseContext(), "custom", Toast.LENGTH_SHORT).show();
                    }
                });
                splashView.addCustomView(linearLayout);
            }
        }

        @Override
	//click
        public void onAdClicked(PANative result) {

        }

        @Override
	//close
        public void onAdClosed(PANative result) {

        }
    });
}

```



## <a name="error">Error Code For SDK</a>

| Error Code                        | Description                              |
| --------------------------------- | ---------------------------------------- |
| ERR\_000\_TRACK                   | Track exception                          |
| ERR\_001\_INVALID_INPUT           | Invalid parameter                        |
| ERR\_002\_NETWORK                 | Network exception                        |
| ERR\_003\_REAL_API                | Error from Ad Server                     |
| ERR\_004\_INVALID_DATA            | Invalid advertisement data               |
| ERR\_005\_RENDER_FAIL             | Advertisement render failed              |
| ERR\_006\_LANDING_URL             | Landing URL failed                       |
| ERR\_007\_TO_DEFAULT_MARKET       | Default Landing URL failed               |
| ERR\_008\_DL_URL                  | Deep-Link exception                      |
| ERR\_009\_DL_URL_JUMP             | Deep-Link jump exception                 |
| ERR\_010\_APK_DOWNLOAD_URL        | Application package download failed      |
| ERR\_011\_APK_INSTALL             | Application install failed               |
| ERR\_012\_VIDEO_LOAD              | Load the video exception                 |
| ERR\_013\_PAGE_LOAD               | HTML5 page load failed                   |
| ERR\_014\_JAR_UPDATE_VERSION      | Update jar check failed                  |
| ERR\_015\_GET_GAID                | Cannot get google advertisement id - GAID retrieval failed |
| ERR\_016\_GET_AD_CONFIG           | Cannot get the account configuration or template |
| ERR\_017\_INTERSTITIAL_SHOW_NO_AD | Tried to load the interstitial advertisement, but the advertisement is not ready/loading |
| ERR\_018\_AD_CLOSED               | Ad slotId has been closed                |
| ERR\_999\_OTHERS                  | All other errors                         |


