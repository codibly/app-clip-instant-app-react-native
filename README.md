# App Clips & Instant Apps


This article describes the process of creating App Clips and Instant Apps for React Native - based applications.

An **App Clip** is a small part of an iOS application that can be downloaded and opened by one of the invocation methods, e.g. QR codes, NFC tags, and App Clip Codes. Apple created them to save users time, such as making quick restaurant orders and payments or renting a scooter without having to search the App Store.

App Clips available on devices running iOS 15 and earlier versions have a 10 MB size limit of uncompressed binary, but for the devices running iOS 16 and later, the uncompressed App Clip binary can be up to 15 MB in size.

An **Instant App** is an application that can be downloaded and run with the help of the **Google Play Instant** service. It can be loaded directly from the Play Store - by tapping the "**Try Now**" button, or it can be invoked similar to an App Clip with one of the methods - e.g. QR code, Deep Link, website banner. This type of Android app is mostly used as a demo version of a game - e.g. Clash Royale, or limited/test versions of apps like RedBullTV.  
Google’s Instant Apps have also a download size limit of **15 MB**.

### App Clip & Instant App

<img src="images/appclipinstantapp.gif" width="450" alt="appclipinstantapp"/>

All sections are tested on the latest(**0.70.3**) version of React Native. I’m also using Typescript in my project.

Agenda:

1.  [How to create a React Native App Clip?](Creating-React-Native-AppClip.md)

2.  [How to create a React Native Instant App?](Creating-React-Native-InstantApp.md)

3.  [Handling size of React Native App Clip](Handling-Size-React-Native-AppClip.md)

4.  [Handling size of React Native Instant App](Handling-Size-React-Native-InstantApp.md)

5.  [Linking in App Clip](Linking-AppClip.md)

6.  [Linking in Instant App](Linking-InstantApp.md)

7.  [Testing App Clip Launch Experience](Testing-AppClip.md)


Repository with completed project - containing iOS App, Android App, App Clip, Instant App: [📚ArticleApp Repo](https://github.com/dawidzawada/ArticleApp)