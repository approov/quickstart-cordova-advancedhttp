# Shapes Example

This quickstart is written specifically for Android and iOS apps that are implemented using [`Cordova`](https://cordova.apache.org) and the [`Cordova Advanced HTTP networking plugin`](https://www.npmjs.com/package/cordova-plugin-advanced-http). This quickstart provides a detailed step-by-step example of integrating Approov into an app using a simple `Shapes` example that shows a geometric shape based on a request to an API backend that can be protected with Approov.

## WHAT YOU WILL NEED
* Access to a trial or paid Approov account
* The `approov` command line tool [installed](https://approov.io/docs/latest/approov-installation/) with access to your account
* [Android Studio](https://developer.android.com/studio) installed (version Hedgehog 2023.1.1 is used in this guide) if you will build the Android app. Note that the `ANDROID_HOME` value must be properly defined to allow the plugin to be built. Note that if you are using Java SDK 17 or later you may experience a compatibility issue with Gradle. In this case you may need to reference an earlier JDK in your `gradle.properties` file as discussed [here](https://stackoverflow.com/questions/70598676/unsupported-class-file-major-version-61-cordova-mac).
* [Android SDK Build Tools](https://developer.android.com/tools) version 33.0.2.
* [Gradle Build Tool](https://gradle.org/install/) installed if you are using Android (7.4.2 was used in this guide)
* [Xcode](https://developer.apple.com/xcode/) installed (version 15.1 is used in this guide) to build iOS version of application
* [iOS Deploy](https://www.npmjs.com/package/ios-deploy) must be installed if intend to run on iOS devices
* [Cocoapods](https://cocoapods.org) installed to support iOS building (1.11.3 used in this guide)
* An iOS device or simulator if you  are using the iOS platform
* An Android device or emulator if you are using the Android platform
* The content of this repo

## RUNNING THE SHAPES APP WITHOUT APPROOV

The app is initially setup to run the standard [`Cordova Advanced HTTP networking plugin`](https://www.npmjs.com/package/cordova-plugin-advanced-http).

Firstly, we need to add the platforms and the standard plugin. Open a shell terminal at the `cordova-advanced-http/shapes-app` directory and type the following:

```
cordova platform add android@12.0.0
```
```
cordova platform add ios@7.0.1
```
```
cordova plugin add cordova-plugin-advanced-http
```

Now we need to build and run the Cordova Shapes App. Type the following command for Android:

```
cordova run android
```

This builds the app for Android and runs it on a connected device.

Running an iOS app on a device requires code signing. Open the Xcode project located in `cordova-advanced-http/shapes-app/platforms/ios/CordovaApproovShapes.xcworkspace`:

```
open platforms/ios/CordovaApproovShapes.xcworkspace
```

Select your code signing team in the `Signing & Capabilities` section of the project. Also ensure you modify the app's `Bundle Identifier` so it contains a unique string (you can simply append your company name). This is to avoid Apple rejecting a duplicate `Bundle Identifier` when code signing is performed. Then return to the shell and enter:

```
cordova run ios
```

You should now be able to use the app to say hello and get shapes. You will see two buttons:

<p>
    <img src="readme-images/cordova-shapes-app-start.png" width="256" title="Shapes App">
</p>

Click on the `Say Hello` button and you should see this:

<a>
    <img src="readme-images/cordova-shapes-app-okay.png" width="256" title="Shapes App Good">
</a>

This checks the connectivity by connecting to the endpoint `https://shapes.approov.io/v1/hello`. Now press the `Get Shape` button and you will see this (or a different shape):

<a>
    <img src="readme-images/cordova-shape-triangle.png" width="256" title="Triangle">
</a>

This contacts `https://shapes.approov.io/v1/shapes` to get the name of a random shape. This endpoint is protected with an API key that is built into the code, and therefore can be easily extracted from the app.

The subsequent steps of this guide show you how to provide better protection, either using an Approov Token or by migrating the API key to become an Approov managed secret.

## ADD THE APPROOV PLUGIN DEPENDENCY

In a shell terminal at the `cordova-advanced-http/shapes-app` directory, type the following commands:

```
cordova plugin remove cordova-plugin-advanced-http
```
```
cordova plugin add @approov/cordova-plugin-advanced-http
```

This removes the standard plugin and installs the Approov capable plugin from [npm](https://www.npmjs.com/).

## ENSURE THE SHAPES API IS ADDED

In order for Approov tokens to be generated or secrets managed for the shapes endpoint, it is necessary to inform Approov about it. Execute the following command:

```
approov api -add shapes.approov.io
```

Note that any Approov tokens for this domain will be automatically signed with the specific secret for this domain, rather than the normal one for your account.

## MODIFY THE APP TO USE APPROOV

Uncomment the three lines of Approov initialization code in `cordova-advanced-http/shapes-app/www/js/index.js`:

```Javascript
cordova.plugin.http.approovInitialize("<enter-your-config-string-here>",
```

The Approov SDK needs a configuration string to identify the account associated with the app. It will have been provided in the Approov onboarding email (it will be something like `#123456#K/XPlLtfcwnWkzv99Wj5VmAxo4CrU267J1KlQyoz8Qo=`). Copy this and use it to replace the text `<enter-your-config-string-here>`.

You should also change the Shapes endpoint the app is using by uncommenting the line:

```Javascript
var SHAPE_URL = "https://shapes.approov.io/v3/shapes";
```

Remember to comment out the previous definition of `SHAPE_URL`.

## ADD YOUR SIGNING CERTIFICATE TO APPROOV

You should add the signing certificate used to sign apps so that Approov can recognize your app as being official.

### Android
Add the local certificate used to sign apps in Android Studio. The following assumes it is in PKCS12 format:

```
approov appsigncert -add ~/.android/debug.keystore -storePassword android -autoReg
```

See [Android App Signing Certificates](https://approov.io/docs/latest/approov-usage-documentation/#android-app-signing-certificates) if your keystore format is not recognized or if you have any issues adding the certificate. This also provides information about adding certificates for when releasing to the Play Store. Note also that you need to apply specific [Android Obfuscation](https://approov.io/docs/latest/approov-usage-documentation/#android-obfuscation) rules when creating an app release.

### iOS
These are available in your Apple development account portal. Go to the initial screen showing program resources:

![Apple Program Resources](readme-images/program-resources.png)

Click on `Certificates` and you will be presented with the full list of development and distribution certificates for the account. Click on the certificate being used to sign applications from your particular Xcode installation and you will be presented with the following dialog:

![Download Certificate](readme-images/download-cert.png)

Now click on the `Download` button and a file with a `.cer` extension is downloaded, e.g. `development.cer`. Add it to Approov with:

```
approov appsigncert -add development.cer -autoReg
```

If it is not possible to download the correct certificate from the portal then it is also possible to [add app signing certificates from the app](https://approov.io/docs/latest/approov-usage-documentation/#adding-apple-app-signing-certificates-from-app).

> **IMPORTANT:** Apps built to run on the iOS simulator are not code signed and thus auto-registration does not work for them. In this case you can consider [forcing a device ID to pass](https://approov.io/docs/latest/approov-usage-documentation/#forcing-a-device-id-to-pass) to get a valid attestation.

## SHAPES APP WITH APPROOV API PROTECTION

Run the app again as follows, for Android:

```
cordova run android
```

For iOS:

```
cordova run ios
```

Press the `Get Shape` button. You should now see this (or another shape):

<a>
    <img src="readme-images/cordova-shape-rectangle.png" width="256" title="Rectangle">
</a>

This means that the app is obtaining a validly signed Approov token to present to the shapes endpoint.

> **NOTE:** Running the app on an Android emulator or iOS simulator will not provide valid Approov tokens. You will need to ensure it always passes on your the device (see below).

## WHAT IF I DON'T GET SHAPES

If you don't get a valid shape then there are some things you can try. Remember this may be because the device you are using has some characteristics that cause rejection for the currently set [Security Policy](https://approov.io/docs/latest/approov-usage-documentation/#security-policies) on your account:

* Ensure that the version of the app you are running is signed with the correct certificate.
* On Android, look at the [`logcat`](https://developer.android.com/studio/command-line/logcat) output from the device. You can see the specific Approov output using `adb logcat | grep ApproovService`. This will show lines including the loggable form of any tokens obtained by the app. You can easily [check](https://approov.io/docs/latest/approov-usage-documentation/#loggable-tokens) the validity and find out any reason for a failure.
* On iOS, look at the console output from the device using the [Console](https://support.apple.com/en-gb/guide/console/welcome/mac) app from MacOS. This provides console output for a connected simulator or physical device. Select the device and search for `ApproovService` to obtain specific logging related to Approov. This will show lines including the loggable form of any tokens obtained by the app. You can easily [check](https://approov.io/docs/latest/approov-usage-documentation/#loggable-tokens) the validity and find out any reason for a failure.
* Use `approov metrics` to see [Live Metrics](https://approov.io/docs/latest/approov-usage-documentation/#metrics-graphs) of the cause of failure.
* You can use a debugger or emulator/simulator and get valid Approov tokens on a specific device by ensuring you are [forcing a device ID to pass](https://approov.io/docs/latest/approov-usage-documentation/#forcing-a-device-id-to-pass). As a shortcut, you can use the `latest` as discussed so that the `device ID` doesn't need to be extracted from the logs or an Approov token.
* Also, you can use a debugger or Android emulator and get valid Approov tokens on any device if you [mark the signing certificate as being for development](https://approov.io/docs/latest/approov-usage-documentation/#development-app-signing-certificates).

## SHAPES APP WITH SECRETS PROTECTION

This section provides an illustration of an alternative option for Approov protection if you are not able to modify the backend to add an Approov Token check.

Firstly, revert any previous change to `cordova-advanced-http/shapes-app/www/js/index.js` for `SHAPE_URL` so that it uses `https://shapes.approov.io/v1/shapes/`, which simply checks for an API key.

The `API_KEY` should also be changed to `shapes_api_key_placeholder`, removing the actual API key out of the code.

We need to inform Approov that it needs to substitute the placeholder value for the real API key on the `Api-Key` header. Find this line and uncomment it:

```Javascript
cordova.plugin.http.approovAddSubstitutionHeader("Api-Key", "");
```

You must inform Approov that it should map `shapes_api_key_placeholder` to `yXClypapWNHIifHUWmBIyPFAm` (the actual API key) in requests as follows:

```
approov secstrings -addKey shapes_api_key_placeholder -predefinedValue yXClypapWNHIifHUWmBIyPFAm
```

> Note that this command requires an [admin role](https://approov.io/docs/latest/approov-usage-documentation/#account-access-roles).

Build and run the app again and press the `Get Shape` button. You should now see this (or another shape):

<a>
    <img src="readme-images/cordova-shape-square.png" width="256" title="Square">
</a>

This means that the app is able to access the API key, even though it is no longer embedded in the app configuration, and provide it to the shapes request.
