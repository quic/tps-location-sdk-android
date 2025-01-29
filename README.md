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

### 5.16.4

* ![feat] ![all] Introduced Wi-Fi RTT positioning
* ![feat] ![all] Added support for GeoFencing API in XPS v2
* ![feat] ![provider] Added support for an additional token-based registration model
* ![impr] ![all] Improved TTFF in tiling-only mode
* ![impr] ![all] Improved performance in XPS tracking mode
* ![opt] ![all] Increased the default Wi-Fi scanning rate when connected to an AP

### 5.15.3

* ![impr] ![all] Updated on-device Wi-Fi positioning algorithm
* ![impr] ![all] Additional configuration options for GNSS fix acquisition
* ![feat] ![all] Added support for passive scanning mode
* ![feat] ![sdk] Street address and timezome lookup support in XPS v2
* ![opt] ![provider] Minor power optimization in location provider
* ![depr] ![all] Removed legacy XPS v1 location engine
* ![depr] ![sdk] Deprecated Tune Location API
* ![depr] ![sdk] Deprecated Certified Location API

### 5.14.13

* ![fix] ![all] Fixed a stability issue when running in 5G/NR cellular mode

### 5.14.12

* ![fix] ![sdk] Fixed handling GPS locations with no satellite info in Z-axis

### 5.14.11

* ![feat] ![provider] Location provider support for customer-specific extention added in 5.4

### 5.14.10

* ![fix] ![all] Fixed handling a rare scenario when telephony service dies

### 5.14.9

* ![fix] ![sdk] Fixed handling GPS location age in Z-axis

### 5.14.8

* ![fix] ![all] Fixed a critical error happening after a cell tile update

### 5.14.7

* ![fix] ![provider] Location provider maintenance release

### 5.14.4

* ![opt] ![all] Optimized wake lock usage in XPS v2

### 5.14.3

* ![opt] ![all] Optimized handling of cached cell scan data
* ![impr] ![all] Improved performance in Wi-Fi/Cell only mode
* ![feat] ![all] Extended configuration capabilities

### 5.14.2

* ![feat] ![provider] Location provider release with SDK 5.14 features

### 5.14.1

* ![fix] ![sdk] Fixed a regression where a stale GPS could be reported in XPS v2
* ![fix] ![sdk] Fixed the Z-axis GPS-only use case
* ![opt] ![all] Added a cache for on-device neighbor cell lookup

### 5.14.0

* ![feat] ![all] On-device cell-based positioning using tiles
* ![feat] ![all] Improved on-device Wi-Fi positioning with cell corroboration

### 5.13

* ![feat] ![sdk] SDK released as a standalone library (AAR)
* ![feat] ![all] XPS v2 is now the default mode
* ![feat] ![all] Location tracking and smoothing in XPS v2
* ![feat] ![all] Added indoor-specific attributes in the API

### 5.12

* ![feat] ![provider] Location provider released as a generic build (APK)

### 5.11

* ![feat] ![sdk] Elevation determination based on barometric pressure (Z-axis)

### 5.10

* ![feat] ![provider] Added support for emergency mode in Android 10+ for location provider releases
* ![fix] ![all] Fixed cell scanning on Android 10+

### 5.9

* ![impr] ![sdk] Improvements in XPS v2 performance
* ![feat] ![sdk] Extended XPS v2 configuration and logging capabilities
* ![feat] ![sdk] Added permissive mode support for custom Android integrations
* ![fix] ![sdk] Fixed performance of cell location cache in legacy Android versions

### 5.8

* ![feat] ![sdk] Background mode (long period) support in XPS v2
* ![feat] ![sdk] Location setting check in XPS v2
* ![impr] ![sdk] Improvements in token-based registration
* ![fix] ![sdk] Added a workaround for reference leak in AlarmManager API

### 5.7

* ![feat] ![sdk] New location engine - "XPS v2"
* ![feat] ![sdk] Added support for using custom network connection
* ![opt] ![all] Improved location cache performance

### 5.6

* ![feat] ![sdk] Updated the location setting check for Android 9+

### 5.5.1

* ![fix] ![provider] Fixed the Opt-In mechanism in location provider

### 5.5.0

* ![feat] ![provider] Core support for token-based registration (for location provider releases)
* ![feat] ![all] Additional flexibility in the location cache configuration

### 5.4

* ![feat] ![sdk] Extended capabilities for a customer-specific application

### 5.3

* ![impr] ![all] WPS positioning improvements for the underground subway use case

### 5.2

* ![opt] ![sdk] Optimized external dependencies and components in the manifest
* ![opt] ![all] Optimized performance of Wi-Fi scanning code
* ![opt] ![all] Fixed cell scan timestamps on Android 10+
* ![feat] ![sdk] Initial BLE support in the SDK core

### 5.0.2

* ![fix] ![all] Fixed a rare deadlock condition

### 5.0.0

* ![doc] Migrated SDK documentation to GitHub
* ![doc] Created the Quick Start guide
* ![feat] ![sdk] Distribution via Maven repository
* ![feat] ![sdk] Added support for vertical positioning error
* ![impr] ![all] Improved security for tile downloads
* ![impr] ![sdk] Tiling directory is now created by the SDK if it didn't exist
* ![opt] ![all] Minor optimization for Lite Doze mode

### 4.10.x

* ![impr] ![provider] Internal improvements required for location provider releases

### 4.9.8

* ![impr] ![sdk] Additional geofencing improvements

### 4.9.7

* ![feat] ![sdk] Added compatibility with Wi-Fi scan throttling in Android 8
* ![impr] ![sdk] Improvements for Android Doze mode
* ![impr] ![sdk] Improved geofencing performance

### 4.9.6

* ![feat] ![sdk] Added compatibility with Lite Doze mode in Android 7

### 4.9.5

* ![feat] ![all] Added client back-off mechanism

### 4.9.4

* ![impr] ![sdk] Improvements in street address lookup

### 4.9.3

* ![opt] ![all] Improved general CPU utilization and power consumption
* ![opt] ![all] Optimized network load and power consumption in tiling mode
* ![opt] ![sdk] Improved geofencing in UMTS and LTE networks

### 4.9.2

* ![feat] ![sdk] Added a check to respect the system-wide location permission settings
* ![impr] ![sdk] Removed the requirement for the calling app to hold the `READ_PHONE_STATE` permission

### 4.9.1

* ![impr] ![all] Improved power consumption
* ![feat] ![sdk] Added support for Android 4.4 Kit-Kat
* ![feat] ![sdk] Added support for Wi-Fi scan-only mode introduced in Android 4.3
* ![feat] ![sdk] The SDK will now throw an exception if the `READ_PHONE_STATE` permission is not held by the calling app

### 4.9.0

* ![feat] ![sdk] Introducing key-based authentication

### 4.8

* ![feat] ![sdk] Added certified location
* ![feat] ![sdk] Two new geofence types `INSIDE` and `OUTSIDE` for cases where immediate triggering is desired
* ![opt] ![sdk] Improved power consumption during geofencing
* ![opt] ![sdk] Fixed an edge case that could result in the client downloading extra tile data
* ![impr] ![sdk] Improved accuracy of location on slow scanning devices

### 4.7.6

* ![fix] ![sdk] Fixed a bug that could negatively affect time to fix on some CDMA devices

### 4.7.5

* ![feat] ![sdk] Added support for location based on LTE cell towers
* ![impr] ![sdk] Improved the accuracy of all cell locations
* ![impr] ![sdk] Improved time to fix and power efficiency when using in-flight location
* ![fix] ![sdk] Fixed a bug in tiling when venue and tuned locations fall on different sides of a tile boundary

### 4.7

* ![feat] ![sdk] Introduced location tuning
* ![feat] ![sdk] Added support for background location updates when the device is asleep
* ![feat] ![sdk] Added support for coarse location based on region codes (known as LACs) provided by the cell network
* ![fix] ![sdk] In-flight positioning fixes

### 4.6

* ![feat] ![sdk] Added in-flight positioning capabilities
* ![impr] ![sdk] Improvements in power consumption and accuracy for geofences
* ![feat] ![sdk] Enhanced indoor location for surveyed sites
* ![feat] ![sdk] Added offline location

### 4.5

Version 4.5 was never released publicly.

### 4.4

* ![feat] ![sdk] Added geofencing
* ![impr] ![sdk] Improved long period support
* ![impr] ![sdk] Improvements to positioning in remote mode

### 4.3

* ![opt] ![sdk] Improved memory usage when downloading tiles

### 4.2

* ![feat] ![sdk] Added auto-registration
* ![impr] ![sdk] Improved offline location coverage

### 4.1

* ![impr] ![sdk] Improved first fix accuracy
* ![impr] ![sdk] Improved location when stationary
* ![impr] ![sdk] Various improvements to hybrid algorithm, particularly in tracking mode

### 4.0

* ![opt] ![sdk] Optimized power management
* ![opt] ![sdk] Better data and bandwidth utilization
* ![impr] ![sdk] Improvements to hybrid positioning
* ![feat] ![sdk] Inertial navigation system

# License
![License](https://img.shields.io/static/v1?label=license&message=CC-BY-ND-4.0&color=green)

Copyright (c) 2023 Qualcomm Innovation Center, Inc. All Rights Reserved.

This work is licensed under the [CC BY-ND 4.0 License](https://creativecommons.org/licenses/by-nd/4.0/). See [LICENSE](LICENSE) for more details.


[sdk]: https://img.shields.io/badge/sdk-blue
[provider]: https://img.shields.io/badge/provider-purple
[doc]: https://img.shields.io/badge/documentation-grey
[all]: https://img.shields.io/badge/provider-purple?style=flat&label=sdk&labelColor=blue
[feat]: https://img.shields.io/badge/feature-cyan
[impr]: https://img.shields.io/badge/improvement-brightgreen
[opt]: https://img.shields.io/badge/optimization-yellow
[fix]: https://img.shields.io/badge/bugfix-orange
[depr]: https://img.shields.io/badge/deprecated-red
