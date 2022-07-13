 [1. Introduction](#chaturmail-ai-email-generator-flutter-code)  
 [2. Architecture](#architecture)  
 [3. Directories](#directories)  
 [4. App Flow](#app-flow)  
 [5. APIs](#app-flow)  
 [6. In-App Notifications](#in-app-notifications)  
 [7. Coins](#coins)  
 [8. Static Assets](#static-assets)  
 [9. Ads](#ads)  
 [10. Tech Stack](#tech-stack)  
 [11. Authors](#authors)  

# ChaturMail: AI Email Generator Flutter Code

ChaturMail is a an AI Email Generator application that generates emails based on Subject and Keywords provided by the user.

The application has premade prompts/templates for emails like Request Email, Announcement Email, Intern Application Email, Job Search Email and many more to ease out the process of generation for the user.

The application has tutorials that guide for excellent use of the application.

Users can view their previously generated emails and incase they want to regenerate the same email with some changes then they can do so.

Users can launch their email application which allows them to send them the generated emails hassle free.

Users can also copy the generated email body.

## Architecture

The application follows the Model View View Model (MVVM) architecture powered with the GetX State Management.

After user login the application fetches email prompts, past generated emails of the user if any and tutorials and stores them locally using the Hive database.

All API calls in the app are Asynchronous, they do not block the main thread and their waiting time is handeled accordingly.

Further on whenever the user opens the application the data is initially loaded from local database and subsequently API calls are made to fetch updated data from the server.

This local storage enables offline functionality of the app wherin the users can see their past generated emails, learn using tutorials and study the available prompts.

API responses are either Success or Failure, both are handeled accordingly.

|  API  | Success | Failure |
| :---: |  :---:  |  :---:  |
| `GetPrompts` | Stores data in `prompts` Hive Box, refreshes UI | Loads data from local storage |
| `GetPastEmails` | Stores data in `pastEmails` Hive Box, refreshes UI | Loads data from local storage |
| `GenerateEmail` | Refresh generated emails list, shows email to user | Shows error to user |
| `GetUserData` | Stores data in `users` Hive Box | Show error message to user |

Above are the main API calls and their behaviours.

**Note**: Fields in API column are not the actual endpoints. They are explained further on.

## Directories

Below are explainations of all the directories in `lib/`

`main.dart`: Start point of the application  
`firebase_options.dart`: Generated by `flutter_fire`, Firebase CLI for Flutter setup.



`views`: Has files of UI screens and widgets of the app. Further has:    
- &emsp; `screens`: Has all full screens displayed in the app
- &emsp; `widgets`: Has reusable widgets that are used across the app


`view_models`: Has files that interact with external services like API's and make data available throughout the app to show on screens.
- &emsp; The files in here are `GetXControllers` that provide data via state management. They have methods that are called on user actions, these methods fetch data from API's and make it available and/or store it locally. They also contain error handling.


`models`: Has models that provide abstract blueprint of the objects that'll be used in the app.
- &emsp; The files here also server as schemas for Hive database boxes.
- &emsp; The files with `.g.dart` are generated by `build_runner` and are schemas for the Hive DB.


`controllers`: Has files that control application interaction with external services. For eg. API Calls
- &emsp; `ads_controller.dart`: Provides Google AdMob Ad Unit id's
- &emsp; `api_communicator.dart`: Has API call functions
- &emsp; `google_login.dart`: Facilitates Google Login
- &emsp; `jwt_token_obtainer.dart`: Fetches JWT Token from Auth Service and stores it for further API communications. All API endpoints need a JWT Tokrn.
- &emsp; `login_checker.dart`: Checks if the user is logged in using any of the available options.
- &emsp; `storage_controller.dart`: SharedPreferences Storage handler. Depreciated. Not Used.
- &emsp; `user_controller.dart`: Has user management functionalities like getting user data, deleting user


`utils`: Has files that provide utility features
- &emsp; `api_status.dart`: Defines `Success` and `Failure` responses objects for API's. Response from API is classified into these 2 types and used throughout the app.
- &emsp; `constants.dart`: Has values that are absolute constants in the app. For eg: API Base URL.
- &emsp; `routes_class.dart`: Has GetX Routes defined for navigation. Currently not in use.
- &emsp; `utils_controller.dart`: A GetX controller that has utility functionalitie like internet connectivity check, showing error dialog

## App Flow

Here's an explainiation of actions happening internally as the user progresses  using the app

**Note**: App API responses are encapsulated as objects of `Success` or `Failure` classes defined in `lib/utils/api_status.dart`  

**Note**: All API calls are HTTP `POST` requests. Explained further on.

1. User opens app and logins via appropriate method. Currently only Google login available.
2. After login is suceeded, `/auth/getJWTToken` API is called. This returns a JWT Token used by app other API's.
3. User is navigated to `DashboardScreen`. There in `initState()` API's are called asynchronously via functions from `view_models` while user is shown loading animations. Following are the API's:
- `/prompt/getAllPrompts`: Returns Email templates/prompts
- `/email/getGeneratedEmails`: Returns all generated emails by user
- `/tutorials/getAllTutorials`: Returns tutorials.
- `/user/getUserData`: Returns user's profile data: Name, Email, Picture etc.
- `misc/getBannerImage`: Returns a banner image in `base64`
4. All of the data received above is store in Hive boxes and is loaded locally everytime when user opens app and API responses are to be received.
5. From here user can navigate anywhere in the app. Following are different scenarios explained:
6. User clicks on an email prompt: User can read information about the email prompt from the tooltip and then proceed to fill in following details to generate email on the `GenerateEmailScreen`:
- `to`: Email address of the receiver
- `Subject`: Subject of the email
- `Keywords`: Comma (,) separated keywords about the email.
The user then clicks the `Generate` button and an API call is made to generate the email. Following is the API:
- `/email/generateEmail`: Takes in all above entered field and returns geerated email.  
 6.1. When response is received, the user is navigated to the the `DisplayGeneratedEmailScreen` screen. From here the user can do following actions:
- Click on floating action button to launch their email app
- Copy the generated email text
- Go back and regenerate the email
7. User clicks on past generated email. From there user can either regenerate same email with some changes or read and/or view their generated email. The user is is either taken to:
- `GenerateEmailScreen`
- `DisplayGeneratedEmailScreen`.
8. User clicks on `Tutorials` in the bottom navigation bar. User is taken to the `TutorialsScreen` where are tutorials are listed. User can click on any tutorial and the user will be taken to `TutorialDetailScreen` to view the full tutorial.
9. User clicks on `Profile` in the bottom navigation bar. User is taken to the profile screen where details shows are as follows:
- Name, Email, Picture
- Coins Used, Coins Available
**Note**: More info on coins further on  

10. User opens sliding Menu. In menu user has following options:
- `Tutorials`: Takes user to `TutorialsScreen`
- `Contact Us`: Leads to an in app form to contact the developer
- `Logout`: Logout from the application. Clears all data stored on device and exits the app
- `Delete Account`: Deletes user account. All data from server is deleted
- `Privacy Policy`: Leads to Privacy Policy of the app
## API's

All API's are HTTP REST operating via the `POST` method.

The API's get data using the `POST` method not the `GET` method, this is done because:
- As the application development progresses the API's might need additional data that might cross the limitations of the `GET` request
- `GET` requests encapsulate all data in the URL, any mishap of the URL might lead an attacker direct access to the resources.
- Encrypting `POST` requests is easier and efficient since the encrypted digest might easily cross `GET`'s limit of 2048 characters
- Server security enforcements: The server the API's are deployed on is high security which blocks `GET` requests as per it's policies

To identify users, the server relies on the JWT token which is passed as `Bearer-Token` in all API calls.

The API calls are done from the `lib/controllers/api_communicator.dart` file. It fetches JWT token from Hive storage and adds to every call.

The responses are encapsulated as objects of `Success` and `Failure` defined in the `lib/utils/api_status.dart` file

API Responses have a fixed format as follows:  
`status`: `0`: Failure, `1`: Success -> Indicate success in operation.  
`message`: String message regarding the operation performed.  
`payload`: Is a list `[]` of objects. Any data being sent by the server will be in form of objects in the list. `null` if no payload is returned.  

**Note**: `payload` will never be empty list `[]`, handle it on basis of `null`.

## In-App Notifications

The app has Firebase Notifications SDK integrated. It can receive notifications sent from `Firebase Cloud Messaging (FCM)`
## Coins

Coins are basic units of spending that are spent each time an email is generated.

Each generated email spends 250 coins, the more the coins, better is the quality of the emails, however currently it's restricted to 250.

Currently there are plans to affiliate with Play-to-Earn metaverses for which the coin economics will be used.
## Static Assets

The `assets` folder in the root of the project contains static assets as follows:
 - `images`: Images used in the app
 - `fonts`: Fonts used in the app
## Ads

The app has Google AdMob integrated that serves ads to users.

Ads are integrated to cover up the costs of hosting and maintenance and to keep the app free
## Tech Stack

**Client:** Flutter, Firebase

**Server:** Node, Express, Docker, NGINX, Kubernetes


Flutter 3.0.4 • channel stable • https://github.com/flutter/flutter.git  
Framework • revision 85684f9300
Engine • revision 6ba2af10bb  
Tools • Dart 2.17.5 • DevTools 2.12.2

Node 16.4.0
## Authors

- [Wilfred Almeida](http://wilfredalmeida.com)

