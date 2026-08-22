# EngRes Hub Play Store Readiness Checklist

Last updated: 2026-08-22

This checklist is based on the current EngRes Hub app setup: Firebase Auth, Firestore, Firebase Storage PDF uploads, saved resources, download tokens, AdMob banner/interstitial/rewarded ads, and account deletion requests.

This is a practical release-prep guide, not legal advice.

## Current App Snapshot

- App name: EngRes Hub
- Package/application ID: com.engreshub.app
- Main audience: English teachers and education professionals
- Account system: Firebase email/password authentication
- Backend/data: Firebase Auth, Cloud Firestore, Firebase Storage
- Ads: Google Mobile Ads SDK / AdMob
- Downloads: PDFs saved to phone Downloads/EngRes Hub
- Account deletion: in-app request flow exists in Profile
- Payments/subscriptions: not active in the current app

## Must Fix Before Production Release

### 1. Replace AdMob test IDs

Current app still uses Google demo/test IDs:

- Android AdMob app ID in AndroidManifest.xml: ca-app-pub-3940256099942544~3347511713
- Banner ad unit: ca-app-pub-3940256099942544/6300978111
- Interstitial ad unit: ca-app-pub-3940256099942544/1033173712
- Rewarded ad unit: ca-app-pub-3940256099942544/5224354917

Before production release:

- Create the app in AdMob.
- Create production ad units for Banner, Interstitial, and Rewarded.
- Replace the demo IDs with your own production IDs.
- During testing with production IDs, use AdMob test devices or AdMob test app mode.
- Do not click production ads during testing.

Official references:

- https://developers.google.com/admob/android/test-ads
- https://support.google.com/admob/answer/9388275

### 2. Add a hosted privacy policy URL

Google Play requires a privacy policy in Play Console and a privacy policy link or text inside the app.

Use `docs/privacy_policy_draft.md` as the draft source, then host it somewhere stable, for example:

- a simple website page
- Firebase Hosting
- GitHub Pages
- Google Sites
- your business/app website

After hosting, add the URL in Play Console and add an in-app Privacy Policy entry, probably in Profile or Settings.

Official references:

- https://support.google.com/googleplay/android-developer/answer/10144311
- https://support.google.com/googleplay/android-developer/answer/10787469

### 3. Account deletion readiness

The app currently lets signed-in users request account deletion from Profile. For Google Play, because the app allows account creation, it must provide both a clear in-app deletion request path and an external web deletion request path.

Before release, make sure the real support/admin process does this after a request:

- delete or disable the Firebase Auth user
- delete the user document in Firestore
- delete saved resources tied to the user
- delete download/token records tied to the user where appropriate
- delete account deletion request after processing or mark it completed
- keep only records you are legally or operationally required to keep

Recommended improvement before final release:

- Add a small admin tool/status workflow for account deletion requests, or document the manual Firebase process clearly.
- Create an external web page or form where users can request account deletion outside the app. This URL is required in Play Console for apps that allow account creation.

Official reference:

- https://support.google.com/googleplay/android-developer/answer/13327111

### 4. Confirm target audience

Recommended Play Console target audience for the current app: adults/teachers.

Reason: the app is branded as English Resources for Teachers and includes ads, account creation, and teacher/admin workflows.

Important caution:

- If you choose child age groups, or if the store listing/app wording appears child-directed, Google Play Families rules may apply.
- If the app targets children or mixed child/adult audiences, ads must follow Families rules, and you may need a neutral age screen and non-personalized ads for children.
- Do not opt into Designed for Families unless the app is genuinely designed and compliant for children.

Official references:

- https://support.google.com/googleplay/android-developer/answer/9893335
- https://developers.google.com/edu/guidelines

### 5. Data Safety form draft answers

Use this as the starting point in Play Console. Recheck after any new SDKs or features.

#### Does the app collect user data?

Yes.

#### Does the app share user data?

Likely yes for ads/analytics-like identifiers handled by Google Mobile Ads SDK. Declare third-party SDK collection/sharing according to Google Mobile Ads/AdMob behavior and your ad configuration.

#### Is data encrypted in transit?

Yes, Firebase/Google services use encrypted network transport. Declare based on your actual Firebase/AdMob configuration.

#### Can users request deletion?

Yes. The app has an in-app account deletion request path in Profile.

#### Account information

Collected:

- Name
- Email address
- User ID

Purpose:

- Account management
- App functionality
- Security/fraud prevention/admin support, if applicable

Required or optional:

- Required for signed-in accounts
- Guest reading works without account creation

#### App activity

Collected:

- Saved resources
- Resource opens/views
- Download events
- Download token balance
- Download token usage
- Rewarded ad progress/counts

Purpose:

- App functionality
- Account management
- Analytics/app improvement, if you use the stats this way

#### Files and documents

Admin uploads PDF resources to Firebase Storage. Ordinary users download/view resources provided by the app. The app does not upload users' personal documents as part of normal teacher-user flow.

#### Device or other IDs

Google Mobile Ads SDK may collect device identifiers or advertising identifiers depending on platform/ad configuration. Reflect the Google Mobile Ads SDK disclosure in Play Console.

#### Location

The app does not intentionally request precise or approximate location.

#### Contacts, SMS, call logs, health, financial data

The app does not intentionally collect these.

### 6. Permissions review

Manifest currently declares:

- INTERNET
- WRITE_EXTERNAL_STORAGE with maxSdkVersion 28

Interpretation:

- INTERNET is needed for Firebase, PDF loading, and ads.
- WRITE_EXTERNAL_STORAGE is only for older Android versions where public Downloads writing needs legacy storage access.
- On Android 10+ the app saves files using MediaStore, so broad storage permission is not used.

Before release:

- Confirm Play Console does not flag storage permission unexpectedly.
- Keep storage permission limited to maxSdkVersion 28.

### 7. Content rating / ads disclosure

Declare:

- App contains ads.
- App does not contain user-generated public content.
- App does not contain user-to-user communication.
- App does not contain purchases/subscriptions in the current version.
- App provides educational English teaching resources.

For content rating:

- Answer based on actual resources uploaded.
- Avoid uploading offensive, adult, violent, or copyrighted material that you do not have rights to distribute.

### 8. Firebase Security Checklist

Before release:

- Keep Firebase Storage uploads restricted to the admin account.
- Keep Firestore rules deployed and tested.
- Confirm guests can only read published resource metadata and files intended for public reading.
- Confirm normal users cannot access admin panel actions.
- Confirm account deletion requests are readable only by the owner/admin.
- Confirm download token transactions cannot be forged by users.

### 9. Final Release Build Checklist

Do this only when the app is ready for upload:

- Replace all AdMob demo IDs.
- Add hosted privacy policy URL in Play Console.
- Add privacy policy access inside the app.
- Complete Data Safety form.
- Complete Ads declaration.
- Complete Target audience and content section.
- Complete Content rating questionnaire.
- Create release signing key / upload key.
- Change release signing config away from debug keys.
- Build Android App Bundle: `flutter build appbundle --release`.
- Test the release build on a real device.
- Confirm no debug/test labels remain except AdMob test-device mode during testing.

## Recommended Next App Changes

1. Add a Profile/Settings row: Privacy Policy.
2. Add an admin deletion-request management view or documented manual process.
3. Move AdMob IDs into a config file with clear TEST vs PRODUCTION values.
4. Add a small release-mode guard so test ad IDs cannot accidentally ship.
5. Create the hosted privacy policy page before Play Console setup.