---
title: Publishing
description: How to set up publishing and build status notifications
weight: 3
---

All generated artifacts can be published to external services. The available integrations currently are email, Slack, Google Play and App Store Connect. It is also possible to publish elsewhere with custom scripts, see the examples [below](../yaml/distribution/#publishing).

## Integrations for publishing and notifications

Codemagic has out-of-the-box support for publishing to the services listed below. Read more about each individual integration and see the configuration examples below.

### Email 

If the build finishes successfully, release notes (if passed) and the generated artifacts will be published to the provided email address(es). If the build fails, an email with a link to build logs will be sent.

    publishing:
      email:
        recipients:
          - name@example.com

### Slack

In oder to set up publishing to Slack, you first need to connect the Slack workspace in **User settings > Integrations > Slack** for personal applications and in **Teams > Your_team > Team integrations > Slack** for team apps.

You can then define the channel where build notifications and artifacts will be sent to. If the build finishes successfully, release notes (if passed) and the generated artifacts will be published to the specified channel. If the build fails, a link to the build logs is published. When you set `notify_on_build_start` to `true`, the channel will be notified when a build starts.

    publishing:
      slack:
        channel: '#channel-name'
        notify_on_build_start: true       # To receive a notification when a build starts

### Google Play

Codemagic enables you to automatically publish your app to the `internal`, `alpha`, `beta` and `production` tracks on Google Play. In order to do so, you will need to set up a service account in Google Play Console and add the JSON key file to your Codemagic configuration file, see how to [create a service account](#creating-a-service-account-in-google-play-console).

    publishing:
      google_play:                        # For Android app
        credentials: Encrypted(...)       # JSON key file for Google Play service account
        track: alpha                      # Name of the track: internal, alpha, beta, production

{{<notebox>}}
The proper way to add your keys in `codemagic.yaml` is to copy the contents of the key file and [encrypt](../building/encrypting) it. Then add the encrypted value into the configuration file.
{{</notebox>}}

#### Creating a service account in Google Play Console

1. In Google Play Console, navigate to **Settings > API access**.

2. Click on the **Create Service Account** button and follow the link to **Google API Console**.

3. In Google API Console, click on the **Create Service Account** button.

4. In step 1, fill in the **Service account details** and click **Create**. The name of the service account will allow you to identify it among other service accounts you may have created.

5. In step 2, click the **Select a role** dropdown menu and choose **Project > Editor** as the role.

6. In step 3, you can leave the fields blank and click **Done**.

7. In the list of created service accounts, identify the account you have just created and click on the menu in the **Actions** column.

8. Click **Create key** and select **JSON** as the key type.

9. Click **Create** and save the key file in a secure location to have it available.

### App Store Connect

Codemagic enables you to automatically publish your iOS app to App Store Connect for beta testing with TestFlight or distributing the app to users via App Store. 

    publishing:
      app_store_connect:                  # For iOS app
        apple_id: name@example.com        # Email address used for login
        password: Encrypted(...)          # App-specific password


    publishing:
      email:
        recipients:
          - name@example.com
      slack:
        channel: '#channel-name'
        notify_on_build_start: true       # To receive a notification when a build starts
      google_play:                        # For Android app
        credentials: Encrypted(...)
        track: alpha
      app_store_connect:                  # For iOS app
        apple_id: name@example.com        # Email address used for login
        password: Encrypted(...)          # App-specific password
      github_releases:
        prerelease: false
        artifact_patterns:
          - app-release.apk
          - '*.aab'

{{<notebox>}}

A prerequisite for Slack publishing is connecting the Slack workspace in **User settings > Integrations > Slack** for personal applications and in **Teams > Your_team > Team integrations > Slack** for team apps.

{{</notebox>}}
<br>

{{<notebox>}}
GitHub releases is available for GitHub repositories only.

Publishing happens only for successful builds triggered on tag creation and is unavailable for manual builds.

Note that using `*` wildcard in the beginning of the pattern requires quotation marks around the pattern, otherwise it will violate the `yaml` syntax.
{{</notebox>}}


### Publishing a Flutter package to pub.dev

In order to get publishing permissions, first you will need to log in to pub.dev locally. It can be done with running `pub publish --dry-run`.
After that `credentials.json` will be generated which you can use to log in without the need of Google confirmation through browser.

`credentials.json` can be found in the pub cache directory (`~/.pub-cache/credentials.json` on MacOS and Linux, `%APPDATA%\Pub\Cache\credentials.json` on Windows)

    - echo $CREDENTIALS | base64 --decode > "$FLUTTER_ROOT/.pub-cache/credentials.json"
    - flutter pub publish --dry-run
    - flutter pub publish -f

### Publishing an app to Firebase App Distribution

If you use a Firebase service, encrypt `google-services.json` as `ANDROID_FIREBASE_SECRET` environment variable for Android
or `GoogleService-Info.plist` as `IOS_FIREBASE_SECRET` for iOS.

    echo $ANDROID_FIREBASE_SECRET | base64 --decode > $FCI_BUILD_DIR/android/app/google-services.json
    echo $IOS_FIREBASE_SECRET | base64 --decode > $FCI_BUILD_DIR/ios/Runner/GoogleService-Info.plist

#### Publishing an app using Firebase CLI

Make sure to encrypt `FIREBASE_TOKEN` as an environment variable. Check [documentation](https://firebase.google.com/docs/cli#cli-ci-systems) for details.

Android

    - |
      # publish the app to Firebase App Distribution
      apkPath=$(find build -name "*.apk" | head -1)
      echo "Found apk at $apkPath"

      if [[ -z ${apkPath} ]]
      then
        echo "No apks were found, skip publishing to Firebase App Distribution"
      else
        echo "Publishing $apkPath to Firebase App Distribution"
        firebase appdistribution:distribute --app <your_android_application_firebase_id> --groups <your_android_testers_group> $apkPath
      fi

iOS

    - |
      # publish the app to Firebase App Distribution
      ipaPath=$(find build -name "*.ipa" | head -1)
      echo "Found ipa at $ipaPath"

      if [[ -z ${ipaPath} ]]
      then
        echo "No ipas were found, skip publishing to Firebase App Distribution"
      else
        echo "Publishing $ipaPath to Firebase App Distribution"
        firebase appdistribution:distribute --app <your_ios_application_firebase_id> --groups <your_ios_testers_group> $ipaPath
      fi

#### Publishing an app with Fastlane

Make sure to encrypt `FIREBASE_TOKEN` as an environment variable. Check [documentation](https://firebase.google.com/docs/cli#cli-ci-systems) for details.

Before running a lane, you should install Fastlane Firebase app distribution plugin

        - |
          # install fastlane-plugin-firebase_app_distribution
          gem install bundler
          sudo gem install fastlane-plugin-firebase_app_distribution --user-install

Then you need to call a lane. This code is similar for Android and iOS.

Android

    - |
      # execute fastlane android publishing task
      cd android
      bundle install
      bundle exec fastlane <your_android_lane>

iOS

    - |
      # execute fastlane ios publishing task
      cd ios
      bundle install
      bundle exec fastlane <your_ios_lane>


#### Publishing an Android app with Gradle

To authorize an application for Firebase App Distribution, use [Google service account](https://firebase.google.com/docs/app-distribution/android/distribute-gradle#authenticate_using_a_service_account).
Encrypt and add to environment variables these credentials (the file is named something like `yourappname-6e632def9ad4.json`) as `GOOGLE_APP_CREDENTIALS`. Specify the filepath in your `build.gradle` in `firebaseAppDistribution` as `serviceCredentialsFile="your/file/path.json"`.

    buildTypes {
        ...
        release {
            ...
            firebaseAppDistribution {
                ...
                serviceCredentialsFile="<your/file/path.json>"
            }
        }

 Note that in case the credentials file is not specified in `firebaseAppDistribution` build type, it will search the filepath in `GOOGLE_APPLICATION_CREDENTIALS` environment variable.

Decode application credentials for Firebase authorization:

    echo $GOOGLE_APP_CREDENTIALS | base64 --decode > $FCI_BUILD_DIR/your/file/path.json

Build the application:

    - |
        # set up local properties
        echo "flutter.sdk=$HOME/programs/flutter" > "$FCI_BUILD_DIR/android/local.properties"
    - flutter packages pub get
    - flutter build apk --release

Call the `gradlew` task for distribution

    - |
        # distribute app to firebase with gradle plugin
        cd android
        ./gradlew appDistributionUploadRelease

{{<notebox>}}

If you didn't specify `serviceCredentialsFile`, you may export it to random location like `/tmp/google-application-credentials.json`

    echo $GOOGLE_APP_CREDENTIALS | base64 --decode > /tmp/google-application-credentials.json

And then export the filepath on the gradlew task

    - |
        # distribute app to firebase with gradle plugin
        export GOOGLE_APPLICATION_CREDENTIALS=/tmp/google-application-credentials.json
        cd android
        ./gradlew appDistributionUploadRelease

{{</notebox>}}

## Examples

More detailed examples about using YAML for code signing and publishing can be found here:

* <a href="https://blog.codemagic.io/distributing-module-yaml/" target="_blank" onclick="sendGtag('Link_in_docs_clicked','distributing-module-yaml')">Native Android project</a>
* <a href="https://blog.codemagic.io/distributing-native-ios-sdk-with-flutter-module-using-codemagic/" target="_blank" onclick="sendGtag('Link_in_docs_clicked','distributing-native-ios-sdk-with-flutter-module-using-codemagic/')">Native iOS project</a>
