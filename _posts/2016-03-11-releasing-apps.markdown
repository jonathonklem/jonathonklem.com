---
layout: post
title: Releasing Apps
date: 2016-03-11 03:13:44.000000000 -05:00
categories:
- development
tags: []
status: publish
type: post
published: true
---
I don't ordinarily do app development, but I recently had a project working with a long-time client that involved creating an Ad-Hoc App so that he could showcase some of his video files on his iPad. There were a lot of tears and headaches involved in figuring out how Apple's certificates and mobile provisioning files work, and it took a ridiculous amount of trial and error, however, I was able to do it the exact way he wanted and BEFORE THE DEADLINE.

This got me thinking about finally getting my feet wet in app development. I had already purchased the iOS developer account, so I figured I might as well try to make use of it. I have been experimenting with react-native, and I plan to do some cool stuff with that, however, for this first experiment I just wanted to focus on the deployment process. So, for the project itself I rigged up some simple app with HTML/CSS/jQuery, threw it at PhoneGap and generated my apk and ipa files.

## Generating Keys
The first challenge was getting the key process right. As with everything, Android made it easier. This post isn't meant to be a comprehensive guide to building apps, but an overview of the release process for Google's Play Store and Apple's App Store.

### Android
- Generate keystore:

`keytool -genkey -v -keystore tip-calculator.keystore -alias tip-calculator -keyalg RSA -keysize 2048 -validity 10000`

From the command line, type the above and that's it. Boom! You have your .keystore file. There are several questions you're asked, but it's the same series of questions you have to answer when generating a CSR.

### iOS
- Generate CSR
Apple is kind enough to tell you how to use keystore to generate the CSR when you visit the create dashboard This gives you a .certSigningRequest file
- After you upload your CSR to the certificate creation portion of the panel you'll receive a .cer file.
- Next add your application id in the developer dashboard here
- Create a new provisioning file
- In this step you need to select the proper application ID (which should have been added in the 3rd step) and certificate file (created in step 2). This will spit out a .mobileProvisioning file.
- Lastly you need to extract the private key out of your .cer file and get a .p12 file. While it seems simple now, this step caused me a lot of pain at first. Easiest way to do it is to drag it in to keychain, then right-click and select "extract". Save the .p12 file somewhere handy

To create your apk file for Android, you'll need your keystore, the alias (this has to be typed in to the form exactly as it was specified in the command or else PhoneGap won't know what to do), and the password for the keystore.

To create your ipa file for iOS you'll need the mobileProvisioning file, the .p12 file, and the password you created when you extracted the .p12 file.

### Publishing the App
Now that your binaries are properly signed, it's time to get those binaries in the app store and play store. When I submitted my test application to Google Play, it was approved within about an hour. I submitted my application to Apple's App Store and 24 hours later I still don't have a response. According to this website, the average response time is about 5 days at the time of this writing.

This is also the stage you'll finally need a Google account. Again, I argue that Google does it better (or atleast in a more 'developer friendly' way) by charging a one time fee of $25. Apple charges $99/year. Google also makes it handy in that you can upload the APK from the dashboard, whereas Apple makes you either use xcode or application loader to upload the IPA.

### Android
- Go to the [developer dashboard](https://play.google.com/apps/publish/) and click "Add New Application"
- Enter a name and either upload the APK first (what I did) or select "prepare store listing"
- Assuming you upload the APK first you'll be presented with a button to upload the file
- In the store listing section you're presented with a few well, marked required fields and the following images
    - 2 phone screenshots
    - 1 high-res screenshot (512x512)
    - 1 featured graphic (1024x500)
- There's also a questionnaire to establish a rating.
- Submit for review

### iOS
- Log in to [itunes connect](https://itunesconnect.apple.com), select "my apps", and click the plus icon and then "New App"
- Enter your new app's name and select the bundle id you specified while creating the key files
- On the first panel select your primary and secondary categories
- Go to the 'pricing and availability' section and select a price for your app
- Next navigate to the "Prepare for submission section"
    - There are several required text fields, just like with Google Play such as Title, Short Description, Description, ...
    - Apple also demands images
    - 1 high-res icon (1024x1024)
    - 4 phone screenshots
- Add the IPA
    - This can be done with either xcode or Application Loader. Once the IPA file is uploaded, you can select it back at the new app dashboard
- Submit for review


