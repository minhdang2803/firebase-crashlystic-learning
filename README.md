# Firebase Crashlystic

An example project that learn and practice about Firebase Crashlystic

# Introduction

A Firebase tool which is lightweight, real-time crash reporter called Crashlytics makes it easier to identify, manage, and resolve stability problems that lower the caliber of your app. By logically categorizing crashes and highlighting the factors that precede them, Crashlytics helps you reduce the amount of time you spend debugging.

To use Firebase Crashlystic, we need to pass throught three steps:
    -  Connect the code base of our application to Firebase
    -  Add the Crashlystic SDK via CocoaPods, Gradle, Pub. Our project is in flutter so we use Pub
    -  The crash reports will be recored in the Firebase Console

# Step by steps to install Firebase Crashlystic
## Install Firebase CLI
To install Firebase CLI, we have 2 method, but we will use npm which is stand for Node Packet Manager to install CLI in order to be use with the Powershell or Windows Terminal.

In any directory, we can turn on a Windows Terminal and run:
```bash
$ npm install -g firebase-tools
```
This command enables the globally available firebase command. If the command fail, we need to change [npm permissions](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally), 

## Login the Firebase and test the CLI command
After sucessfully installing the Firebase CLI, we need to authenticate
```bash
$ firebase login
```
> **NOTE:**  
The firebase login command connects to localhost on your computer by opening a web page. Run the command with the flag **—no-localhost** if you're using a remote machine and don't have access to localhost.

To test the CLI is properly installed, we can list all the firebase projects that created by our account:

```bash
$ firebase projects:list
```
## Initialize a Firebase Project
A project directory is necessary for many routine CLI operations, such as deploying to a Firebase project. A project directory is created using the firebase init command. A firebase.json configuration file can be found in a project directory, which is typically the same directory as your source control root.

```bash
$ firebase init
```
At the end of initialization, Firebase automatically creates the following two files at the root of your local app directory:

- A firebase.json configuration file that lists your project configuration.

- A .firebaserc file that stores your project aliases.

> **_NOTE:_** 
The command will overwrite the relevant part of the firebase.json file back to the service's default configuration if you run firebase init for any Firebase service again.

## Deploy a Firebase project
The Firebase CLI manages deployment of code and assets to your Firebase project, including:

- New releases of your Firebase Hosting sites
- New, updated, or existing Cloud Functions for Firebase
- Rules for Firebase Realtime Database
- Rules for Cloud Storage for Firebase
- Rules for Cloud Firestore
- Indexes for Cloud Firestore

To deploy to a Firebase project, run the following command from your project directory:

```bash
$ firebase deploy
```

We can also contain the message each time we deploy the firebase to server: 
```bash
firebase deploy -m "Deploying the best new feature ever."
```

> **_NOTE_:** 
    - To deploy resources from a project directory, the project directory must have a firebase.json file. This file is automatically created for you by the firebase init command.
    - By default, firebase deploy creates a release for all deployable resources in your project directory. To deploy specific Firebase services or features, use [partial deployment](https://firebase.google.com/docs/cli#partial_deploys).

## Get started with Firebase Crashlystic
This quickstart explains how to connect the Crashlytics Flutter plugin with Firebase Crashlytics in our project so that we may receive thorough crash reports in the Firebase console.

We must use both your IDE and a command-line tool to set up Crashlytics. We must force a test exception to be thrown in order to send your first crash report to Firebase and complete setup.

### Before we start
1. Setup a Firebase Project with our Flutter project
2. To get features like crash-free users, breadcrumb logs, and velocity alerts, you need to enable Google Analytics in your Firebase project. Goto **Go to settings > Project settings > Integrations tab**, then follow the on-screen instructions for Google Analytics.

### Step 1: Add Crashlystic to our Flutter Project
1. In the root directory of our Flutter Project, now you can use the command:

```bash
flutter pub add firebase_crashlytics
```
2. Then, run this command to ensure that our Flutter app's Firebase configuration is up-to-date and, for Android, adds the required Crashlytics Gradle plugin to our app. Everytime when we add a firebase packages, we should use this command.

```bash
flutterfire configure
```
3. After ensuring that the Firebase configuration is up-to-date, we can now run the Flutter project in debug mode:

```bash
flutter run
```
### Step 2: Handle our error from Flutter application
1. Import some necessary libraries:
```Dart
// For Connecting Firebase Project to our Flutter Project
import 'firebase_options.dart';
import 'package:firebase_core/firebase_core.dart';

// For Adding Firebase Crashlystic plugin
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

```

2a. By overriding ```FlutterError.onError``` with ```FirebaseCrashlytics.instance.recordFlutterFatalError```, we can catch all the errors generated (thrown) in the Flutter framework

```Dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
  );

  // Pass all uncaught errors from the framework to Crashlytics.
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

  runApp(const MyApp());
}
```
2b. If we want to catch the errors that are not generated by Flutter framework, we can use **zones**


```Dart
//...
import 'dart:async'; //Adding this line

void main() async {
  runZonedGuarded<Future<void>>(() async {
    WidgetsFlutterBinding.ensureInitialized();
  
    await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
    );

    FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

    runApp(MyApp());
  }, (error, stack) =>
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true));
}
```

### Step 3: Do an example crash to finish setting up process
1. Add code to your app that you can use to force a test exception to be thrown.
    - If you’ve added an error handler that calls FirebaseCrashlytics.instance.recordError(error, stack, fatal: true) to the top-level Zone, you can use the following code to add a button to your app that, when pressed, throws a test exception:

```Dart
TextButton(
    onPressed: () => throw Exception(),
    child: const Text("Throw Test Exception"),
),
```

2. Build and run application
3. Force the test exception to be thrown in order to send your app's first report:
    - Open your app from your test device or emulator.
    -   In your app, press the test exception button that you added using the code above.
4. Go to the [Crashlytics dashboard](https://accounts.google.com/ServiceLogin/signinchooser?passive=1209600&osid=1&continue=https%3A%2F%2Fconsole.firebase.google.com%2Fproject%2F_%2Fcrashlytics&followup=https%3A%2F%2Fconsole.firebase.google.com%2Fproject%2F_%2Fcrashlytics&flowName=GlifWebSignIn&flowEntry=ServiceLogin) of the Firebase console to see your test crash.
    -   If you've refreshed the console and you're still not seeing the test crash after five minutes, enable debug logging to see if your app is sending crash reports

## Customize our crash reports using Crashlytics APIs
