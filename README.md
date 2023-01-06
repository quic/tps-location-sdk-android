# TPS Location SDK (Android)
![Platform](https://img.shields.io/static/v1?label=platform&message=android&color=informational)

## Prerequisites

Supported on all Android versions starting with 2.3.x (Gingerbread).

Contact support.tps@qti.qualcomm.com to request access to the SDK, obtain the API key or get additional information.

## Installation

### Add SDK to your project

Put the SDK library file under the `libs/` subdirectory and add it to the `dependencies` section of your `build.gradle`:
```gradle
dependencies {
    implementation files('libs/wpsapi.aar')
}
```

### Permissions

Location SDK automatically adds the following permissions to your app's manifest:

| Android Permission                         | Used For
|--------------------------------------------|-----------------------------------------------------------
| android.permission.INTERNET                | Communication with Qualcomm's location servers
| android.permission.CHANGE_WIFI_STATE       | Initiation of Wi-Fi scans
| android.permission.ACCESS_WIFI_STATE       | Obtaining information about the Wi-Fi environment
| android.permission.ACCESS_COARSE_LOCATION  | Obtaining Wi-Fi or cellular based locations
| android.permission.ACCESS_FINE_LOCATION    | Accessing GPS location for hybrid location functionality
| android.permission.WAKE_LOCK               | Keeping processor awake when receiving background updates
| android.permission.ACCESS_NETWORK_STATE    | Checking network connection type to optimize performance

### Background Mode

Starting with **Android 8**, the system enforces significant limitations in location determination capabilities when the app is running in background, for power consumption and user privacy considerations.

In order to use the Location SDK in background mode, a regular app has to provide general means of background execution - e.g. by running a foreground service, or setting up a periodic alarm. A foreground service is required to achieve location update rates faster than once every 30 minutes.

Add the following service declaration in your manifest if you are planning to use the Location SDK in background without running a foreground service:
```xml
<service
    android:name="com.skyhookwireless.spi.network.NetworkJobService"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:exported="false"/>
```

The following receiver declaration is recommended (but not required) if your app's minimum supported Android version is below **API 24 (Nougat)**:
```xml
<receiver
    android:name="com.skyhookwireless.spi.power.AlarmReceiver"
    android:exported="false"/>
```

If your app is targeting **API level 29 (Android 10)** or higher and you are willing to determine location in background, add the following permission to your manifest file:

| Android Permission                            | Used For
|-----------------------------------------------|----------------------------------
| android.permission.ACCESS_BACKGROUND_LOCATION | Determine location in background

### Using the Android Emulator

The Precision Location SDK will not be able to determine location using Wi-Fi or cellular beacons from the emulator because it is unable to scan for those signals. Because of that, its functionality will be limited on the emulator. In order to verify your integration of the SDK using the emulator, you may want to use the `getIPLocation()` method call. The full functionality will work only on an Android device.

### Android System Settings

The Skyhook SDK will respect the system setting with regard to location. This includes the granular location settings for GPS and network based location. The SDK will return `WPS_ERROR_LOCATION_SETTING_DISABLED` if location cannot be determined because of the current location setting. XPS will continue to determine location when only the GPS location setting is enabled whereas WPS requires that network based location is enabled.

## Initializing

### Import the SDK

Import the Skyhook WPS package:
```java
import com.skyhookwireless.wps;
```

### Initialize API

Create an instance of [XPS](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/XPS.html) (or [WPS](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html)) API object in the `onCreate` method of your activity or service:
```java
private IWPS xps;
...
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    xps = new XPS(this);
    ...
}
```

Call [setKey()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#setKey-java.lang.String-) to initialize the API with your key:
```java
xps.setKey("YOUR KEY");
```

If you use a key and a SKU label for authentication with TPS, call [setAuthentication()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#setAuthentication-java.lang.String-java.lang.String-) instead as shown below:
```java
xps.setAuthentication("YOUR KEY", "YOUR SKU");
```

### Enable tiling mode

If you are planning to request location determination frequently, it is recommended to enable the tiling mode in the SDK. It downloads, from the server, a small portion of the database so the device can automonously determine its location, without further need to contact the server.

This mode is activated by calling [setTiling()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#setTiling-java.lang.String-long-long-com.skyhookwireless.wps.TilingListener-):
```java
File tileDir = new File(getFilesDir(), "tiles");

xps.setTiling(
    tileDir.getAbsolutePath(),  // directory where to store tiles
    9 * 50 * 1024,              // 3x3 tiles (~50KB each) for each session
    9 * 50 * 1024 * 10,         // total size is 10x times the session size
    null);
```

### Request location permission

With the runtime permissions model introduced in **Android M**, the Precision Location SDK requires location permission to be granted before calling most of its methods. Depending on how the SDK is used in the application, developer can decide when to request the permission and if an explanation needs to be displayed for the user:
```java
ActivityCompat.requestPermissions(
    this, new String[] { Manifest.permission.ACCESS_FINE_LOCATION }, 0);
```

In order to determine location in background on **Android 10** or higher, make sure to check that the user has granted the "Allow all the time" (`ACCESS_BACKGROUND_LOCATION`) location permission.

## Location API

### One-time location request

The one-time location request provides a method to make a single request for location: [getLocation()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#getLocation-com.skyhookwireless.wps.WPSAuthentication-com.skyhookwireless.wps.WPSStreetAddressLookup-boolean-com.skyhookwireless.wps.WPSLocationCallback-).

The request will call the callback method. The [handleWPSLocation()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPSLocationCallback.html#handleWPSLocation-com.skyhookwireless.wps.WPSLocation-) method is passed the [WPSLocation](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPSLocation.html) object which will contain the location information.

A one-time location request call from your application would look like this:
```java
xps.getLocation(null, WPSStreetAddressLookup.WPS_NO_STREET_ADDRESS_LOOKUP, false, new WPSLocationCallback() {
    @Override
    public void handleWPSLocation(WPSLocation location) {
        // Do something with location
    }

    @Override
    public WPSContinuation handleError(WPSReturnCode error) {
        // ...

        // To retry the location call on error use WPS_CONTINUE,
        // otherwise return WPS_STOP
        return WPSContinuation.WPS_CONTINUE;
    }

    @Override
    public void done() {
        // after done() returns, you can make more WPS calls
    }
});
```

You can also request a street address or time zone lookup with this method.

### Periodic location

A request for periodic location updates can be made using the following method: [getPeriodicLocation()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#getPeriodicLocation-com.skyhookwireless.wps.WPSAuthentication-com.skyhookwireless.wps.WPSStreetAddressLookup-boolean-long-int-com.skyhookwireless.wps.WPSPeriodicLocationCallback-).

This method will continue running for the specified number of iterations or until the user stops the request by returning [WPS_STOP](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPSContinuation.html#WPS_STOP) from the callback.

It is highly recommended to enable [tiling mode](#enable-tiling-mode) with this location method to eliminate frequent network requests to the server. Note that if street address or time zone lookup is requested, this call will **not use the tiling cache** to determine locations.

Starting with **Android Pie** the recommended minimum period for location updates is **30 seconds or longer**.

### Offline location

The API supports offline location that allows the application to determine the location of the device even offline and outside of tile coverage by collecting a token that can be replayed when the device is once again online. Offline tokens are only valid for 90 days after they are generated. Attempting to redeem a token more than 90 days old will result in an error.

* [getOfflineToken()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#getOfflineToken-com.skyhookwireless.wps.WPSAuthentication-byte:A-)
* [getOfflineLocation()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#getPeriodicLocation-com.skyhookwireless.wps.WPSAuthentication-com.skyhookwireless.wps.WPSStreetAddressLookup-boolean-long-int-com.skyhookwireless.wps.WPSPeriodicLocationCallback-)

### Abort

When your app is terminating, it is recommended to cancel any ongoing operations by invoking the [abort()](https://quic.github.io/tps-location-sdk-android/javadoc/com/skyhookwireless/wps/WPS.html#abort--) method:
```java
public void onDestroy() {
    super.onDestroy();
    xps.abort();
    ...
}
```

## API reference

Check the [full API reference](https://quic.github.io/tps-location-sdk-android/javadoc) for more information on APIs that are exposed by the SDK.

## Logging

To turn logging on, add the following code in the `onCreate()` method of your activity, service or application:
```java
SharedPreferences.Editor skyhookPrefs = getSharedPreferences("skyhook", MODE_PRIVATE).edit();
skyhookPrefs.putBoolean("com.skyhook.wps.LogEnabled", true);
skyhookPrefs.apply();
```

By default, the SDK will output log messages to Android logcat.

If you want the SDK to write log to a file, set the following preferences:
```java
skyhookPrefs.putBoolean("com.skyhook.wps.LogEnabled", true);
skyhookPrefs.putString("com.skyhook.wps.LogType", "BUILT_IN,FILE");
skyhookPrefs.putString("com.skyhook.wps.LogFilePath", "/sdcard/wpslog.txt");
skyhookPrefs.apply();
```

If you want to write the log file to external storage, make sure to obtain the `WRITE_EXTERNAL_STORAGE` permission in your app.

## Changelog

### 5.13

* SDK released as a standalone library (AAR)
* Location tracking and smoothing in XPS v2
* Added indoor-specific attributes in the API

### 5.12

* Location provider released as a generic build (APK)

### 5.11

* Elevation determination based on barometric pressure (Z-axis)

### 5.10

* Added support for emergency mode in Android 10+ for location provider releases
* Fixed cell scanning on Android 10+

### 5.9

* Improvements in XPS v2 performance
* Extended XPS v2 configuration and logging capabilities
* Added permissive mode support for custom Android integrations
* Fixed performance of cell location cache in legacy Android versions

### 5.8

* Background mode (long period) support in XPS v2
* Location setting check in XPS v2
* Improvements in token-based registration
* Added a workaround for reference leak in AlarmManager API

### 5.7

* New location engine - "XPS v2"
* Added support for using custom network connection
* Improved location cache performance

### 5.6

* Updated the location setting check for Android 9+

### 5.5.1

* Fixed the Opt-In mechanism in location provider

### 5.5.0

* Core support for token-based registration (for location provider releases)
* Additional flexibility in the location cache configuration

### 5.4

* Extended capabilities for a customer-specific application

### 5.3

* WPS positioning improvements for the underground subway use case

### 5.2

* Optimized external dependencies and components in the manifest
* Optimized performance of Wi-Fi scanning code
* Fixed cell scan timestamps on Android 10+
* Initial BLE support in the SDK core

### 5.0.2

* Fixed a rare deadlock condition

### 5.0.0

* Migrated SDK documentation to GitHub
* Created the Quick Start guide
* Distribution via Maven repository
* Added support for vertical positioning error
* Improved security for tile downloads
* Tiling directory is now created by the SDK if it didn't exist
* Minor optimization for Lite Doze mode

### 4.10.x

* Internal improvements required for location provider releases

### 4.9.8

* Additional geofencing improvements

### 4.9.7

* Added compatibility with Wi-Fi scan throttling in Android 8
* Improvements for Android Doze mode
* Improved geofencing performance

### 4.9.6

* Added compatibility with Lite Doze mode in Android 7

### 4.9.5

* Added client back-off mechanism

### 4.9.4

* Extended the street address lookup with a new geospatial layer which represents a smaller township or municipal names found within a larger city region. This layer is sourced from open-source data without modification and more closely aligns with general jurisdictional and town boundaries.

### 4.9.3

* Improved general CPU utilization and power consumption
* Optimized network load and power consumption in tiling mode
* Improved geofencing in UMTS and LTE networks

### 4.9.2

* Added a check to respect the system-wide location permission settings
* Removed the requirement for the calling app to hold the `READ_PHONE_STATE` permission

### 4.9.1

* Improved power consumption
* Added support for Android 4.4 Kit-Kat
* Added support for Wi-Fi scan-only mode introduced in Android 4.3
* The SDK will now throw an exception if the `READ_PHONE_STATE` permission is not held by the calling app

### 4.9.0

* Introducing key-based authentication

### 4.8

* Added certified location
* Improved power consumption during geofencing
* Two new geofence types `INSIDE` and `OUTSIDE` for cases where immediate triggering is desired
* Fixed an edge case that could result in the client downloading extra tile data
* Improved accuracy of location on slow scanning devices

### 4.7.6

* Fixed a bug that could negatively affect time to fix on some CDMA devices

### 4.7.5

* Added support for location based on LTE cell towers
* Improved the accuracy of all cell locations
* Improved time to fix and power efficiency when using in-flight location
* Fixed a bug in tiling when venue and tuned locations fall on different sides of a tile boundary

### 4.7

* Introduced location tuning
* Added support for background location updates when the device is asleep
* Added support for coarse location based on region codes (known as LACs) provided by the cell network
* In-flight positioning fixes

### 4.6

* Added in-flight positioning capabilities
* Improvements in power consumption and accuracy for geofences
* Enhanced indoor location for surveyed sites
* Added offline location

### 4.5

Version 4.5 was never released publicly.

### 4.4

* Added geofencing
* Improved long period support
* Improvements to positioning in remote mode

### 4.3

* Improved memory usage when downloading tiles

### 4.2

* Added auto-registration
* Improved offline location coverage

### 4.1

* Improved first fix accuracy
* Improved location when stationary
* Various improvements to hybrid algorithm, particularly in tracking mode

### 4.0

* Optimized power management
* Better data and bandwidth utilization
* Improvements to hybrid positioning
* Inertial Navigation System

# License
![License](https://img.shields.io/static/v1?label=license&message=CC-BY-ND-4.0&color=green)

Copyright (c) 2023 Qualcomm Innovation Center, Inc. All Rights Reserved.

This work is licensed under the [CC BY-ND 4.0 License](https://creativecommons.org/licenses/by-nd/4.0/). See [LICENSE](LICENSE) for more details.
