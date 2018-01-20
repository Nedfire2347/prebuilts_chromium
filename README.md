# README

## DOWNLOAD

- Android Jelly Bean and newer: [Download ChromePublic.apk](https://github.com/andi34/prebuilt_chromium/raw/master/ChromePublic.apk)
- Android Lollipop and newer: [Download ChromeModernPublic.apk](https://github.com/andi34/prebuilt_chromium/raw/master/ChromeModernPublic.apk)
- Android Nougat and newer: [Download MonochromePublic.apk](https://github.com/andi34/prebuilt_chromium/raw/master/MonochromePublic.apk)


## Changes on top of official Chromium Source

- enable dubious Do Not Track feature by default
- disable navigation error correction by default
- disable contextual search by default
- disable network prediction by default
- disable metrics by default
- disable hyperlink auditing by default
- disable autofill by default
- disable first run welcome page
- disable data reduction proxy promotions
- mark non-secure origins as non-secure
- add DuckDuckGo search engine
- stop enabling search engine geolocation by default
- use Chromium branding in more places
- hide passwords.google.com link when not supported
- Chromium: use startpage searchengine by default
- Chromium: disable strict mode


## Information for Developer

You can use this repo to add Chromium prebuilt into your Rom. Please read below information.


### How to add Chromium prebuilt into our Rom?

Add Chromium to your build config once you have cloned this repo into your Rom source:

```
# Chromium Browser
PRODUCT_PACKAGES += \
    Chromium

```
(e.g. inside your device.mk)


On most roms you can avoid adding Chromium to your device config.
Most Roms support an easy way to add more packages - you only need to create
```
vendor/extra/product.mk
```
and add call the PRODUCT_PACKAGES from there.

Here's an example of an local_manifest you can use:
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <remote  name="andi34"
           fetch="https://github.com/andi34/" />

  <project path="vendor/extra" name="android_vendor_extra" remote="andi34" revision="m" />
  <project path="prebuilts/chromium-browser" name="prebuilts_chromium" remote="andi34" revision="master" />

</manifest>
```

CyanogenMod initial added support for an easy way to add more packages by the following changes:

1. [add a hook to add extra packages](https://github.com/LineageOS/android_vendor_cm/commit/45e6598078e3a919ec644b8ad22bf9ad2912e876)
2. [cm: Update use of vendor/extra](https://github.com/LineageOS/android_vendor_cm/commit/b9dd400b385b18519cad768b5ec45fcbe9973f22)
3. [Allow vendor/cyngn overlays to override vendor/cm overlays](https://github.com/LineageOS/android_vendor_cm/commit/39b9aa83f003aaf9842bc091ef0a4c2d3d2d15e4)

You can adapt those changes to your rom source if it isn't supported yet.


### Let's avoid patching our device tree or rom source to add Chromium prebuilt to our Rom

- For Android 5 and below we are using ChromePublic.apk
- For Android 6 and newer we are using ChromeModernPublic.apk


### How to use MonochromePublic on Android 7+

To use prebuilt MonochromePublic.apk on Android 7+ you need to add chromium to frameworks/base/core/res/res/xml/config_webview_packages.xml
(Your can also add it to your device overlay).

```
<?xml version="1.0" encoding="utf-8"?>
<!-- Copyright 2015 The Android Open Source Project
     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
          http://www.apache.org/licenses/LICENSE-2.0
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<webviewproviders>
    <!-- The default WebView implementation -->
    <webviewprovider description="Chromium" packageName="org.chromium.chrome" availableByDefault="true">
    </webviewprovider>
</webviewproviders>
```
Once that's done you need to [modify the Android.mk](https://github.com/andi34/prebuilts_chromium/commit/09cd63b824ffa08c2365d276d1540ba45cf3c865#diff-3ae6be565f1e33e90e0b11f768de1f6c) to use MonochromePublic.apk.


### How to use ChromeModernPublic on Android 5

On Android 5 ChromeModernPublic.apk can't be used as prebuilt without patching your Rom source:

Run these commands:
```
cd build
curl https://raw.githubusercontent.com/andi34/prebuilt_chromium/master/patches/Lollipop/0001-Fix-Chrome.patch | git am -
```
Once that's done you need to modify the Android.mk to use ChromeModernPublic.apk:
```
LOCAL_SRC_FILES    := ChromeModernPublic.apk
LOCAL_PREBUILT_JNI_LIBS_arm := @lib/armeabi-v7a/crazy.libchrome.so
LOCAL_PREBUILT_JNI_LIBS_arm += @lib/armeabi-v7a/libchromium_android_linker.so
```


### Multiple Chrome APK Targets according to the official build instructions

1. `chrome_public_apk` (ChromePublic.apk)
   * `minSdkVersion=16` (Jelly Bean).
   * Stores libchrome.so compressed within the APK.
   * Uses [Crazy Linker](https://cs.chromium.org/chromium/src/base/android/linker/BUILD.gn?rcl=6bb29391a86f2be58c626170156cbfaa2cbc5c91&l=9).
   * Shipped only for Android < 21, but still works fine on Android >= 21.
2. `chrome_modern_public_apk` (ChromeModernPublic.apk)
   * `minSdkVersion=21` (Lollipop).
   * Uses [Crazy Linker](https://cs.chromium.org/chromium/src/base/android/linker/BUILD.gn?rcl=6bb29391a86f2be58c626170156cbfaa2cbc5c91&l=9).
   * Stores libchrome.so uncompressed within the APK.
     * This APK is bigger, but the installation size is smaller since there is
       no need to extract the .so file.
3. `monochrome_public_apk` (MonochromePublic.apk)
   * `minSdkVersion=24` (Nougat).
   * Contains both WebView and Chrome within the same APK.
     * This APK is even bigger, but much smaller than SystemWebView.apk + ChromePublic.apk.
   * Stores libchrome.so uncompressed within the APK.
   * Does not use Crazy Linker (WebView requires system linker).
     * But system linker supports crazy linker features now anyways.
