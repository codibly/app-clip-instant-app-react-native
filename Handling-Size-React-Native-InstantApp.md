# Handling size of React Native Instant App

Google limits the size of the Instant Android App Bundle to **15 MB** before any kind of compression - that's not a lot for React Native based applications - but how to measure the size of our app bundle - we've got a tool for that provided with **Android Studio**.

Let’s go to our `scripts` folder and create a new bash script called - `measure_instant_app.bash`, paste this code inside:

```
# Measure size of instant bundle - value is returned in bytes
bundletool get-size total --apks=local_app.apks --instant
```

Alright, before running our new script, we need to build our Instant App:

```
./scripts/build_run_instant_app.bash
```

After a successful build, we can measure the size of our app with our new script:

```
./scripts/measure_instant_app.bash
```

That’s my result:

```
MIN,MAX
5594175,5594175
```

The size of our React Native Instant App is `5.6 MB` from the start - this might be different in your case, depending on your React Native version.

Currently we're including all native packages provided by the RN team. If we would like to add another npm package that uses native code and use it in components imported into our **Main App** - the size of the **Instant App** would increase anyway.

It’s because of a feature introduced in **React Native 0.60** - **auto-linking**. But what can we do to reduce the size of the application bundle?

Well, there’s a possibility to disable **auto-linking** and choose only packages that are required for the **Instant App** to work, but of course, we want to keep **auto-linking** for our **Main App** - it is a useful feature.

Let’s start by adding a new Java class to our `appInstant` module. Open **Android Studio**, choose **Android View**, open `appInstant module -> java -> right-click on the com.articleapp directory`.

`New -> Java Class` - name this class `PackageListInstant`.

```
package com.articleapp;

import android.app.Application;
import android.content.Context;
import android.content.res.Resources;
import com.facebook.react.ReactNativeHost;
import com.facebook.react.ReactPackage;
import com.facebook.react.shell.MainPackageConfig;
import com.facebook.react.shell.MainReactPackage;
import java.util.ArrayList;
import java.util.Arrays;

public class PackageListInstant {
    private Application application;
    private ReactNativeHost reactNativeHost;
    private MainPackageConfig mConfig;

    public PackageListInstant(ReactNativeHost reactNativeHost) {
        this(reactNativeHost, null);
    }

    public PackageListInstant(Application application) {
        this(application, null);
    }

    public PackageListInstant(ReactNativeHost reactNativeHost, MainPackageConfig config) {
        this.reactNativeHost = reactNativeHost;
        mConfig = config;
    }

    public PackageListInstant(Application application, MainPackageConfig config) {
        this.reactNativeHost = null;
        this.application = application;
        mConfig = config;
    }

    private ReactNativeHost getReactNativeHost() {
        return this.reactNativeHost;
    }

    private Resources getResources() {
        return this.getApplication().getResources();
    }

    private Application getApplication() {
        return this.application;
    }

    private Context getApplicationContext() {
        return this.getApplication().getApplicationContext();
    }

    public ArrayList<ReactPackage> getPackages() {
        return new ArrayList<>(Arrays.<ReactPackage>asList(
                new MainReactPackage(mConfig)
        ));
    }
}
```

Basically, we will use this class to access packages chosen by us instead of **auto-linked** ones. Notice that the `getPackages()` method returns `MainReactPackage`. In the future, you will be adding more packages to this list.

Alright, let's continue. The next step would be to use our class in the `MainApplication.java` file. Version `0.68` introduced a new architecture for React Native - currently it is optional to use it, but even if you are not using it for your project, the new architecture has a `MainApplication` equivalent file called `MainApplicationReactNativeHost.java` - we need to make the same changes in both files.

Open both mentioned Java classes and find this import:

```
import com.facebook.react.PackageList;
```

Remove it from both files, then look for the `@Override` of method `protected List<ReactPackage> getPackages()` - remove the code inside and replace it with:

```
@Override
protected List<ReactPackage> getPackages() {
    // Packages need to be linked manually due to requirements of instant app
    // If the dependency is needed in an instant application - import and append it with the property path in PackageListSyntheticFull class and include the package in build.gradle file
    
    return new PackageListInstant(this).getPackages();
}
```

Additionally, add this import only in `MainApplicationReactNativeHost.java`:

```
import com.articleapp.PackageListInstant;
```

Finally, there is one more change to make - open `build.gradle` of `appInstant` and find this line:

```
apply from: file("../../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesAppBuildGradle(project)
```

Basically, this line is importing **auto-linking** functionality. Remove this line. Now let’s build and run the instant app with `build_run_instant_app.bash` and run the `measure_instant_app.bash` script again. Here’s my result:

```
MIN,MAX
5594475,5594475
```

The size is basically the same, but now we are controlling which libraries will be appended to our **Instant App**. Let’s find out how to add new packages now. Start by adding 2 new scripts - this time for our full app.

Create a new bash script file called `build_run_full_app.bash` with the code below:

```
#!/bin/bash

set -e

adb uninstall com.articleapp || true

# clean
sh scripts/cleanup.bash
./gradlew clean

# build bundle
./gradlew app:bundleRelease

# build apks file from bundle based on connected device configuration
bundletool build-apks --bundle=app/build/outputs/bundle/release/app-release.aab  --output=local_app.apks --connected-device --ks=debug.keystore --ks-pass=pass:12345678 --ks-key-alias=key0 --key-pass=pass:12345678

# install full app
bundletool install-apks --apks local_app.apks

# run full app
adb shell am start -n com.articleapp/com.articleapp.MainActivity
```

And the second script for measuring size `measure_full_app.bash` with the code below:

```
# Measure size of Main App Android Bundle - value is returned in bytes
bundletool get-size total --apks=local_app.apks
```

Let’s build our full app and measure how much space it takes:

```
MIN,MAX
7110343,7110343
```

The size of our Main App is `7.1 MB`, our **Instant App** size currently takes `5.6 MB`. Let's test how much our sizes will increase for both after we add 2 new packages, open the root project folder and run the command:

```
yarn add @react-native-firebase/app @react-native-firebase/messaging
```

These 2 packages are used mainly for connecting apps to Firebase & managing notifications from the Firebase service. Build and run the **Main App** once again and measure the size:

```
MIN,MAX
7690722,7690722
```

In my case, the size of the Main App increased to almost `7.7 MB`. Let's measure our **Instant App** now:

```
MIN,MAX
5620122,5620122
```

The size is still almost the same. It is because of disabling **auto-linking** in our **appInstant module** - new native libraries are simply not appended to our module - of course, we cannot use them right now. Time to add them to our linked packages list. Open the `build.gradle` file of appInstant and add these imports in the dependencies object:

```
implementation project(":react-native-firebase_app")
implementation project(":react-native-firebase_messaging")
```

My gradle file dependencies object looks like this:

```
dependencies {
    implementation fileTree(dir: "libs", include: ["*.jar"])

    // Manually linked packages:
    implementation project(":react-native-firebase_app")
    implementation project(":react-native-firebase_messaging")
    
    //noinspection GradleDynamicVersion
    implementation "com.google.android.gms:play-services-instantapps:18.0.1"
    implementation "com.facebook.react:react-native:+"  // From node_modules
    // ...
}
```

Notice that the name of the import is different from the `yarn` command name of the package. Import names and instructions for manual linking installation can be found sometimes in libraries documentation - sometimes it's good to look at **README** history before auto-linking was introduced. Alright, now it is time to add our 2 packages in the `PackageListInstant` class. Open the PackageListInstant.java file and add these two imports at the top:

```
import io.invertase.firebase.app.ReactNativeFirebaseAppPackage;
import io.invertase.firebase.messaging.ReactNativeFirebaseMessagingPackage;
```

Then, scroll down to the `getPackages()` method and insert these new package instances:

```
new ReactNativeFirebaseAppPackage(),
new ReactNativeFirebaseMessagingPackage(),
```

My `PackageListInstant` class looks like this now:

```
// Other imports...

// Manually added packages:
import io.invertase.firebase.app.ReactNativeFirebaseAppPackage;
import io.invertase.firebase.messaging.ReactNativeFirebaseMessagingPackage;

public class PackageListInstant {
    // Class stuff...
    
    public ArrayList<ReactPackage> getPackages() {
        return new ArrayList<>(Arrays.<ReactPackage>asList(
                new MainReactPackage(mConfig),
                new ReactNativeFirebaseAppPackage(),
                new ReactNativeFirebaseMessagingPackage()
        ));
    }
}
```

Save your changes and build and run your app - app should be built successfully and work after adding your packages(and providing additional changes if they are required for given packages). My size-measurement results for **Instant App** are:

```
MIN,MAX
5808581,5808581
```

The size has increased to around `5.8 MB` and newly added packages can be used inside the **Instant App**.

Next steps:

[Linking in App Clip](Linking-AppClip.md)

[Linking in Instant App](Linking-InstantApp.md)