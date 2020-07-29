# Content
* [Scene-Based](#Scene-Based)     
  * [Intro](#Intro)     
  * [State transitions](#State-transitions)     
* [App-Based](#App-Based)     
  * [Intro](#Intro)     
  * [State transtitions](#State-transtitions)     
* [Details for app-base life-cycle](#Details-for-app-base-life-cycle)     
  * [Load app into foreground](#Load-app-into-foreground)     
  * [Load app into background](#Load-app-into-background)     
  * [Respond interrupt](#Respond-interrupt)     
  * [Move into background](#Move-into-background)     
  * [Return back foreground](#Return-back-foreground)     

# Scene-Based
## Intro
* If your app supports scenes, UIKit delivers separate life-cycle events for each;
* A scene represents one instance of your app's UI running on a device;
* The user can create multiple scenes for each app, and show and hide them separately;
* Each scene has its own life-cycle, each can be in a different state of execution;
* One scene might be in the foreground while others are in the background or are suspended;

## State transitions
![scene-based state transitions](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/scene-based%20state%20transitions.png)
* When the user or system requests a new scene for your app, UIKit creates it and puts it in the unattached state;
* User-requested scenes move quickly to the **foreground**, where they appear onscreen;
* A system-requested scene typically moves to the background so that it can process an event;

----
**[For example]**
* The system might launch the scene in the background to process a location event;
*  When the user dismisses your app's UI, UIKit moves the associated scene to the background state and eventually to the suspended state;
*  UIKit can disconnect a background or suspended scene at any time to reclaim its resources, returning that scene to the unattached state;


# App-Based
## Intro
* In iOS 12 and earlier, and in apps that don't support scenes, UIKit delivers all life-cycle events to the **UIApplicationDelegate**  object;
* The app delegate manages all of your app's windows, including those displayed on separate screens;
* As a result, app state transitions affect the app's entire UI, including content on external displays;

## State transtitions
![app-based state transitions](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/app-based%20state%20transitions.png)
* After launch, the system puts the app in the inactive or background state, depending on whether the UI is about to appear onscreen;
* When launching to the foreground, the system transitions the app to the active state automatically;
* After that, the state fluctuates between active and background until the app terminates;

# Details for app base life cycle
## Load app into foreground 
![load app into foreground](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/load%20app%20into%20foreground.png)

## Load app into background
![load app into background](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/load%20app%20into%20background.png)

## Respond interrupt
![something interrupts then into inactive](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/something%20interrupts%20then%20into%20inactive.png)

## Move into background
![move into background](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/move%20into%20background.png)

## Return back foreground
![return back foreground](https://github.com/iii17-grace/AppleDocumentation/blob/master/Images/return%20back%20foreground.png)


