# ApkAnalyzer
Java app / library used to obtain detailed informations about Andoid APK files.
* [How to use it](#How_to_use_it)
  * [Prepare for first use](#Prepare_for_first_use)
  * [Analyze APKs](#Analyze_APKs)
  * [Compare APKs](#Compare_APKs)
* [Collected data](#Collected_data)
  * [Attributes & description](#ad)
   * [Basic apk metadata](#basic)
   * [Android manifest metadata](#manifest)
   * [Certificate metadata](#certificate)
   * [Resources metadata](#resource)
   * [File hashes](#hash)
* [Example of analyze result json](#json_analyze)
* [Example of compare result json](#json_compare)
* [Example of statistics result json](#json_statistics)
* [Used libraries](#Used_libs)
 

<a name="How_to_use_it"/>
## How to use it
<a name="Prepare_for_first_use"/>
#### Prepare for first use
If you want to build and make changes in code of ApkAnalyzer, you need to perform following steps.
ApkAnalyzer uses Apktool for decompilation of Apk. You need to add Apktool to your maven repository.<br/>
1. Download apktool_2.0.0rc4 from https://bitbucket.org/iBotPeaches/apktool/downloads<br/>
2. Run following maven command : mvn install:install-file -Dfile=<path-to-apktool_2.0.0rc4-file> -DgroupId=ApkTool -DartifactId=ApkTool -Dversion=2.0.0.rc4 -Dpackaging=jar<br/>

ApkAnalyzer uses maven to build. It`s designed to allow you to customize way it works and various values as thresholds. This can only be done using public API, not using command line parameters (not implemented yet).

You can also use [latest build jar with all dependecies](https://github.com/MartinStyk/ApkAnalyzer/blob/master/jar/ApkAnalyzer-1.0-SNAPSHOT-jar-with-dependencies.jar) and run it as a java program. 
###### Command line parameters
Parameter |Info
------------- | -------------
-analyze      | Triggers analyze task. See [Analyze APKs] (#Analyze_APKs) chapter
-compare      | Triggers compare task. See [Compare APKs] (#Compare_APKs) chapter
-statistics   | Triggers statistics task. See [Statistics] (#Statistics) chapter
-in, --input-dir   | Specify directory where input for task will be searched 
-out, --output-dir   | Specify directory where output of task will be saved 


<a name="Analyze_APKs"/>
#### Analyze APKs 
In case you use jar file, this use case can be triggered with following command ``java -jar -analyze -in="your_input_dir" -out="your_output_dir"``


This task unzip and decompile APK file using ApkTool. To find details about implementation, please explore [AnalyzeTask.java] (https://github.com/MartinStyk/ApkAnalyzer/blob/master/src/main/java/sk/styk/martin/bakalarka/execute/tasks/AnalyzeTask.java).  

This task creates json file for every analyzed APK. [See example of output file](#json_analyze).

<a name="Compare_APKs"/>
#### Compare APKs 
In case you use jar file, this use case can be triggered with following command ``java -jar -compare -in="your_input_dir" -out="your_output_dir"``

Directory ``your_input_dir`` must contain json files created by [analyze task](#Analyze_APKs)

Directory ``your_output_dir`` will contain data about similar APKs


This task only compares metadata. It uses informations about number of activities, services, broadcast recevers, content prividers, apk file size, dex and arsc file size to determine whether two APKs are at least similar. If so, it compares all files in APKs. Default threshold is set to 50% for each attribute. It can not be adjusted using CLI so far. 
In case you need to adjust it for your use, please feel free to see [CompareTask.java] (https://github.com/MartinStyk/ApkAnalyzer/blob/master/src/main/java/sk/styk/martin/bakalarka/execute/tasks/CompareTask.java) and related parts of code. 

Output of this task is json file for every pair of similar APKs. Output is divided into specific folders according to certificate match and version of application match. Every json contains simple diff of two APKs with data including modified, added or deleted files. See [single output file](#json_compare).

<a name="Statistics"/>
#### Statistics 
In case you use jar file, this use case can be triggered with following command ``java -jar -statistics -in="your_input_dir" -out="your_output_dir"``

Directory ``your_input_dir`` must contain json files created by [analyze task](#Analyze_APKs)

Directory ``your_output_dir`` will contain statistics data

To find details about implementation, please explore [StatisticsTask.java] (https://github.com/MartinStyk/ApkAnalyzer/blob/master/src/main/java/sk/styk/martin/bakalarka/execute/tasks/StatisticsTask.java).  

[See example of output file](#json_statistics).

<a name="Collected_data"/>
##Collected data

<a name="ad"/>
###Attributes & description

<a name="basic"/>
####Basic apk metadata
Name          | Type     | Description
------------- | -------- | ------------------------------ 
fileName      | String   | Name of analyzed apk file  
sourceOfFile  | String   | Location from where the file was downloaded
fileSize      | Long     | Size of whole apk file (in bytes)
dexSize       | Long     | Size of compiled sources in classes.dex file (in bytes)
arscSize      | Long     | Size of compiled resources in classes.dex file (in bytes)

<a name="manifest"/>
####Android manifest metadata
Name          | Type     | Description
------------- | -------- | ------------------------------ 
packageName      | String   | [See Android documentation](http://developer.android.com/guide/topics/manifest/manifest-element.html#package)
versionCode  | String   | [See Android documentation](http://developer.android.com/guide/topics/manifest/manifest-element.html#vcode) 
installLocation      | String     | [See Android documentation](http://developer.android.com/guide/topics/manifest/manifest-element.html#install)
numberOfActivities       | Integer     | Total number of activities in application
numberOfServices      | Integer     | Total number of services in application
numberOfContentProviders      | Integer   | Total number of content providers in application  
numberOfBroadcastReceivers  | Integer   | Total number of broadcast receivers in application
usesPermissions      | List<String>     | List of permissions used by application [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-permission-element.html)
usesLibrary       | List<String>     | List of libraries used by application [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-library-element.html)
permissions       | List<String>     | List of permissions defined by application [See Android documentation](http://developer.android.com/guide/topics/manifest/permission-element.html)
permissionsProtectionLevel       | List<String>     | Protection level of permissions defined by application [See Android documentation](http://developer.android.com/guide/topics/manifest/permission-element.html)
usesFeature      | List<String>     | List of features used by application [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-feature-element.html)
usesMinSdkVersion      | String   | Minimum Sdk version required by app List of features used by application [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html)
usesTargetSdkVersion  | String   | Target Sdk version required by app [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html)
usesMaxSdkVersion      | String     | Maximal Sdk version requered by app [See Android documentation](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html)
supportsScreensResizeable       | Boolean     |  [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)
supportsScreensSmall      | Boolean     | [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)
supportsScreensNormal      | Boolean   | [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)
supportsScreensLarge  | Boolean   | [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)
supportsScreensXlarge      | Boolean     | [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)
supportsScreensAnyDensity       | Boolean     | [See Android documentation](http://developer.android.com/guide/topics/manifest/supports-screens-element.html)

<a name="certificate"/>
####Certificate metadata
Name          | Type     | Description
------------- | -------- | ------------------------------ 
fileName      | String   | Name of certificate file (i.e CERT.RSA in MEATA-INF directory)  
signAlgorithm  | String   | Signature algorithm name from the certificate
signAlgorithmOID      | String     | Signature algorithm OID string from the certificate
startDate       | Date     | notBefore date from the validity period of the certificate
endDate      | Date     | notAfter date from the validity period of the certificate
publicKeyMd5  | String   | MD5 hash of public key
certBase64Md5      | String | Base64 MD5 hash of certificate
certMd5       | String | MD5 hash of certificate
version      | Integer | Version value from the certificate
issuerName   | String | Representation of the X.500 distinguished name using the format defined in RFC 2253
subjectName   | String | Representation of the X.500 distinguished name using the format defined in RFC 2253

<a name="resource"/>
####Resources metadata
Name          | Type     | Description
------------- | -------- | ------------------------------ 
locale      | List<String>   | Localizations of string.xml file
numberOfStringResource  | Integer   | Number of entries in default string.xml file (currently not collected)
pngDrawables      | Integer     | Number of drawables in png format
ninePatchDrawables      | Integer     | Number of drawables in .9.png format
jpgDrawables       | Integer     | Number of drawables in jpg/jpeg format
gifDrawables      | Integer     | Number of drawables in gif format
xmlDrawables  | Integer   | Number of drawables in xml format
ldpiDrawables      | Integer | Number of drawables located in ldpi folder
mdpiDrawables       | Integer | Number of drawables located in mdpi folder
hdpiDrawables      | Integer | Number of drawables located in hdpi folder
xhdpiDrawables   | Integer | Number of drawables located in xhdpi folder
xxhdpiDrawables      | Integer | Number of drawables located in xxhdpi folder
xxxhdpiDrawables       | Integer | Number of drawables located in xxxhdpi folder
tvdpiDrawables       | Integer | Number of drawables located in tvdpi folder
nodpiDrawables       | Integer | Number of drawables located in nodpi folder
unspecifiedDpiDrawables      | Integer | Number of drawables located in default drawable folder
rawResources   | Integer | Number of resources in raw/ folder
layouts   | Integer | Total number of layout resources in res/layout* folder
differentLayouts   | Integer | Number of different layout resources in res/layout* folder
menu   | Integer | Total number of menu resources in res/menu folder

<a name="hash"/>
####File hashes
Name          | Type     | Description
------------- | -------- | ------------------------------ 
dexHash      | String | Hash of classes.dex from META-INF/MANIFEST.MF
arscHash      | String | Hash of resources.arsc from META-INF/MANIFEST.MF
drawableHash      | Map<String,String> | Hashes of files in res/drawable* folder from META-INF/MANIFEST.MF. Map<HashValue, fileName>
layoutHash      | Map<String,String> | Hashes of files in res/layout* folder from META-INF/MANIFEST.MF. Map<HashValue, fileName>
otherHash      | Map<String,String>   | Hashes of all files in apk from META-INF/MANIFEST.MF. Map<HashValue, fileName>

<a name="json_analyze"/>
### Example of output *.json
  
  ```json
 {
  "fileName": "com.facebook.katana-42.0.0.27.114-APK4Fun.com.apk",
  "sourceOfFile": "test - works)",
  "fileSize": 35488821,
  "dexSize": 510148,
  "arscSize": 8129808,
  "androidManifest": {
    "packageName": "com.facebook.katana",
    "installLocation": "auto",
    "numberOfActivities": 257,
    "numberOfServices": 84,
    "numberOfContentProviders": 25,
    "numberOfBroadcastReceivers": 60,
    "usesPermissions": [
      "android.permission.ACCESS_COARSE_LOCATION",
      "android.permission.WAKE_LOCK",
      "android.permission.VIBRATE",
      "android.permission.READ_CONTACTS",
      "android.permission.WRITE_CONTACTS",
      "android.permission.GET_ACCOUNTS",
      "android.permission.MANAGE_ACCOUNTS",
      "android.permission.AUTHENTICATE_ACCOUNTS",
      "android.permission.READ_SYNC_SETTINGS",
      "android.permission.WRITE_SYNC_SETTINGS",
      "android.permission.ACCESS_FINE_LOCATION",
      "android.permission.BROADCAST_STICKY",
      "com.facebook.katana.provider.ACCESS",
      "com.facebook.orca.provider.ACCESS",
      "com.facebook.pages.app.provider.ACCESS",
      "android.permission.DOWNLOAD_WITHOUT_NOTIFICATION",
      "android.permission.CAMERA",
      "android.permission.RECORD_AUDIO",
      "android.permission.WRITE_EXTERNAL_STORAGE",
      "com.facebook.permission.prod.FB_APP_COMMUNICATION",
      "com.facebook.permission.prod.SYSTEM_COMMUNICATION",
      "android.permission.READ_PHONE_STATE",
      "android.permission.READ_CALENDAR",
      "android.permission.WRITE_CALENDAR",
      "android.permission.MODIFY_AUDIO_SETTINGS",
      "android.permission.READ_PROFILE",
      "android.permission.READ_SMS",
      "android.permission.CHANGE_NETWORK_STATE",
      "android.permission.CHANGE_WIFI_STATE",
      "android.permission.SYSTEM_ALERT_WINDOW",
      "com.google.android.providers.gsf.permission.READ_GSERVICES",
      "android.permission.READ_EXTERNAL_STORAGE",
      "android.permission.ACCESS_NETWORK_STATE",
      "com.facebook.katana.permission.CROSS_PROCESS_BROADCAST_MANAGER",
      "android.permission.BATTERY_STATS",
      "android.permission.ACCESS_WIFI_STATE",
      "com.android.launcher.permission.INSTALL_SHORTCUT",
      "android.permission.GET_TASKS",
      "android.permission.RECEIVE_BOOT_COMPLETED",
      "android.permission.EXPAND_STATUS_BAR",
      "android.permission.REORDER_TASKS",
      "android.permission.CALL_PHONE",
      "android.permission.SET_WALLPAPER",
      "android.permission.SET_WALLPAPER_HINTS",
      "com.facebook.receiver.permission.ACCESS",
      "android.permission.INTERNET",
      "com.sec.android.provider.badge.permission.READ",
      "com.sec.android.provider.badge.permission.WRITE",
      "com.htc.launcher.permission.READ_SETTINGS",
      "com.htc.launcher.permission.UPDATE_SHORTCUT",
      "com.sonyericsson.home.permission.BROADCAST_BADGE",
      "com.facebook.home.permission.WRITE_BADGES",
      "com.google.android.launcher.permission.READ_SETTINGS",
      "com.google.android.c2dm.permission.RECEIVE",
      "com.facebook.katana.permission.C2D_MESSAGE",
      "com.nokia.pushnotifications.permission.RECEIVE",
      "com.facebook.katana.permission.RECEIVE_ADM_MESSAGE",
      "com.amazon.device.messaging.permission.RECEIVE"
    ],
    "usesLibrary": [
      "com.google.android.maps",
      "com.amazon.device.messaging"
    ],
    "usesFeature": [
      "android.hardware.camera",
      "android.hardware.telephony",
      "android.hardware.microphone",
      "android.hardware.location",
      "android.hardware.location.network",
      "android.hardware.location.gps"
    ]
  },
  "certificateDatas": [
    {
      "fileName": "CERT.RSA",
      "signAlgorithm": "MD5withRSA",
      "signAlgorithmOID": "1.2.840.113549.1.1.4",
      "startDate": "Aug 31, 2009 11:52:16 PM",
      "endDate": "Sep 25, 2050 11:52:16 PM",
      "publicKeyMd5": "57e4647d1da90ef6eef980770673862c",
      "certBase64Md5": "bc45583156a00f55537903de8e84e9d2",
      "certMd5": "3fad024f2dcbe3ee693c96f350f8e376",
      "version": 1,
      "issuerName": "CN=Facebook Corporation,OU=Facebook,O=Facebook Mobile,L=Palo Alto,ST=CA,C=US"
    }
  ],
  "resourceData": {
    "locale": [],
    "numberOfStringResource": 7866,
    "pngDrawables": 3470,
    "jpgDrawables": 4,
    "gifDrawables": 1,
    "xmlDrawables": 1528,
    "differentDrawables": 4935,
    "ldpiDrawables": 1,
    "mdpiDrawables": 62,
    "hdpiDrawables": 3309,
    "xhdpiDrawables": 46,
    "xxhdpiDrawables": 0,
    "xxxhdpiDrawables": 0,
    "unspecifiedDpiDrawables": 1585,
    "rawResources": 85,
    "menu": 32,
    "layouts": 3072,
    "differentLayouts": 3042
  },
  "fileDigest": {
    "dexHash": "bMyeEVy9g8VYPd8S09xBeDu1ouE=",
    "arscHash": "br+poZYwb/JUSmFAKYhW0zd8QVc=",
    "drawableHash": [
      "S6vxCIQ2bS3ze7pSOF+6eozEldc=",
      "uNOeQAFoOFkQbYgJfkJyePznmy0=",
      "727pL5pYYaMczcG0OKDMaYT1F9Q=",
      "u5TY6PQerEhaOucVzSCrKc5lor8=",
      "MuSnK3Cf2lk/cT6NzwWZLRD1O58=",
      "3Qsa4NCFZyy9zsstBa88EvNlvSQ=",
      "uL4rEHyudY8XyvhT2UxSsSfBq8s=",
      "Oykfw3UheEEts18yb9NpJVZtjSE=",
      "xAt0x2WW8LlNCz3IIuearX9FqHc=",
    ],
    "layoutHash": [
      "uNOeQAFoOFkQbYgJfkJyePznmy0=",
      "727pL5pYYaMczcG0OKDMaYT1F9Q=",
      "u5TY6PPerEhaOucVzSCrKc5lor8=",
      "MuSnK3Cf2lk/cT6NzwWZLRD1O58=",
      "3Qsa4NCFZyy9zsstBa88EvNlvSQ=",
      "uL4rEHyudY8XyvhT2UxSsSfBq8s=",
      "Oykfw3UheEEts18yb9NpJVZtjSE=",
      "xAt0x2WW8LlNCz3IIuearX9FqHc=",
     ]
    "otherHash": [
      "aNOeQAFoOFkQbYgJfkJyePznmy0=",
      "927pL5pYYaMczcG0OKDMaYT1F9Q=",
      "q5TY6PQerEhaOucVzSCrKc5lor8=",
      "quSnK3Cf2lk/cT6NzwWZLRD1O58=",
      "cQsa4NCFZyy9zsstBa88EvNlvSQ=",
      "bL4rEHyudY8XyvhT2UxSsSfBq8s=",
      "qykfw3UheEEts18yb9NpJVZtjSE=",
      "yAt0x2WW8LlNCz3IIuearX9FqHc=",
     ]

  }
}
  ```
<a name="Used_libs"/>
### Used libraries
* [xgouchet/AXML](https://github.com/xgouchet/AXML) used code from this project, located in fr.xgouchet.axml package
* [iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)
