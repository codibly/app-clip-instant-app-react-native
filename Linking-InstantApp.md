# Linking in Instant App

Instant Apps, just like App Clips, can be launched by one of the invocation methods - the most popular ones are **deep links** and **QR codes**. When an **Instant App** is launched like this - it receives a **URL** - just like in App Clip, we can pass some information to an app. Instant Apps are similar to standard Android Apps - basically linking works just like in standard Android applications - thanks to that we can use the **Linking Module of React Native** instead of creating a new one just like in the App Clip case.

To configure linking for **Instant App**, follow the [🤖 Create App Links for Instant Apps | Android Developers](https://developer.android.com/training/app-links/instant-app-links) article.

Instant App can also be launched easily without a **URL** - e.g. thanks to the **Try Now** button available on the Play Store - make sure to also handle this case in your Instant App!