
NOTE: Apple is shutting down the iAd App Network on June the 30th. 
Some articles say it does not affect developers trying to integrate iAds but developers that want to have their own apps advertised. 
Some other articles say it will completly shut down. 
Other articles say it will shut down but relaunch using an automated services to host ads.

I am not sure if you can still submit apps or what will happen to the APIs. So for now I included the original project with iAds and another one without iAds.
In the future when I am 100% sure what will happen with iAds I will include another 3rd party ad provider (most likely RevMob and/or Chartboost)
https://developer.apple.com/news/?id=01152016a&1452895272

# iAds, AdMob and CustomAds Helper

A helper class that should make integrating Ads from Apple and Google as well as your own custom Ads a breeze. This helper has been made while designing my SpriteKit game but it can be used for any kind of app. 

The cool thing is that iAds will be used when they are supported otherwise AdMob will be used. iAds tend to have a better impressions and are usually prefered as default ads.
Whats really cool is that if iAd banners are having an error it will automatically load an AdMob banner and if that AdMob banner is than having an error it will try loading an iAd banner again. 
When an iAd inter ad fails it will try an AdMob Inter ad but incase that adMob inter ad also fails it will not try iAd again because you obviously dont want a full screen ad showing at the wrong time.

This Helper creates whats called a shared Banner which is the recommended way by apple. The usual way to achieve this is to put the iAd and adMob banner properties into the appDelegate but because this helper is a Singleton there is no need for this because there is only 1 instance of the class and therefore the banner properties anyway. To read more about shared banner ads you can read this documentation from Apple
https://developer.apple.com/library/ios/technotes/tn2286/_index.html

# Set-Up

- Step 1: Set-Up "-D DEBUG" custom flag. This will reduce the hassle of having to manually change the google ad ids when testing or when releasing. This step is important as google ads will otherwise not work automatically.
This is a good idea in general for other things such as hiding print statements.
Go to Targets (left project sideBar, right at the top) -> BuildSettings. Than underneath buildSettings next to the search bar on the left there should be buttons called Basic, All, Combined and Level. 
Click on All and than you should be able to scroll down in buildSettings and find the section called SwiftCompiler-CustomFlags. Click on other flags and than debug and add a custom flag named -D DEBUG (see the sample project or http://stackoverflow.com/questions/26913799/ios-swift-xcode-6-remove-println-for-release-version)

- Step 2: Copy the Ads.swift file into your project

- Step 3: Copy the Google framework folder found in the sample project into your projects folder on your computer. Its best to copy it to your projects root folder because if you just reference the file (next Step) from a random location on your computer it could cause issues when that file gets deleted/moved. You can download the latest version from Googles website (https://developers.google.com/admob/ios/download)

- Step 4: Add the Google framework to your project. Go to Targets -> BuildPhases -> LinkedBinaries and click the + button. Than press the "Add Other" button and search your computer for the folder you copied at Step 3 containing the googleframework file and add that file. Your linkedBinaries should now say 1.

NOTE: - If you ever update the frameworks, you will need delete the old framework from your project siderbar and the framework folder from your projects root folder on your computer. You need to than go to Targets-BuildSettings-SearchPaths and under FrameworkSearchPaths you should see a link to your old folder. Delete this link to ensure there are no warnings when you add an updated version and than redo step 4.

- Step 5: Add the other frameworks needed. Click the + button again and search for and than add each of these frameworks: AdSupport, AudioToolbox, AVFoundation, CoreGraphics, CoreMedia, CoreTelephony, EventKit, EventKitUI, MessageUI, StoreKit, SystemConfiguration. (https://developers.google.com/admob/ios/quick-start?hl=en
 ). 
You might want to consider putting all the added frameworks you now see in your projects sidebar into a folder called Frameworks, similar to the sample project, to keep it clean.

- Step 6: In your ViewController write the following in ```ViewDidLoad``` before doing any other app set-ups. 
```swift
Ads.sharedInstance.presentingViewController = self
```

- Step 7: This Step is only needed if your app supports both portrait and landscape orientation. Still in your ViewController add the following method.
```swift
override func viewWillTransitionToSize(size: CGSize, withTransitionCoordinator coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransitionToSize(size, withTransitionCoordinator: coordinator)
        
        coordinator.animateAlongsideTransition({ (UIViewControllerTransitionCoordinatorContext) -> Void in
            
            Ads.sharedInstance.orientationChanged()
            
            //let orientation = UIApplication.sharedApplication().statusBarOrientation
            //switch orientation {
            //case .Portrait:
            //    print("Portrait")
            //    // Do something
            //default:
            //    print("Anything But Portrait")
            //    // Do something else
            //}
            
            }, completion: { (UIViewControllerTransitionCoordinatorContext) -> Void in
                print("Device rotation completed")
        })
    }
```
NOTE: This is an ios 8 method, if your app supports ios 7 or below you maybe want to use something like a
```swift
NSNotificationCenter UIDeviceOrientationDidChangeNotification Observer
```

# How to use

There should be no more errors in your project now and the Helper is ready to be used. You can blame Google for most of the work here. 

- iAds are always shown by default unless they are not supported. If you want to manually test Google ads comment out this line in the init method,
```swift
iAdsAreSupported = iAdTimeZoneSupported()
```

- To show a supported Ad simply call these anywhere you like in your project.
```swift
Ads.sharedInstance.showBannerAd() 
Ads.sharedInstance.showBannerAd(withDelay: 1) // delay showing banner slightly eg when transitioning to new scene/view
Ads.sharedInstance.showInterAd()
Ads.sharedInstance.showInterAd(randomness: 4) // 25% chance of showing inter ads (1/4)
```

By default the helper does not include custom ads, if you would like to include your own ads than you simply go the line where you init the helper in your GameViewController (Step 6). Than add this line after setting the presentingViewController property

```swift
Ads.sharedInstance.includeCustomInterAds(totalCustomAds: 2, interval: 4)
```
to include custom ads as well. 
In this example there is 2 custom ads in total. The interval means that every 4th time an Inter ad is shown it will show a custom one, randomised between the totalCustomAds. To add more custom adds go to the struct CustomAds and add more. Than go to 

    showInterAd() { ....
   
and add more cases to the switch statement to match your total custom Ads you want to show. If you dont use custom ads you can comment out the whole block of code after the 2 guard statements at the beginning of the method.

- To remove Banner Ads, for example during gameplay 
```swift
Ads.sharedInstance.removeBannerAds() 
```

- To remove all Ads, mainly for in app purchases simply call 
```swift
Ads.sharedInstance.removeAllAds() 
```

NOTE: - This method will set a removedAds bool to true in the Ads.swift helper. This ensures you only have to call this method to remove Ads and afterwards all the "Ads.sharedInstance.show..." methods will not fire anymore and therefore require no further editing.

For permanent storage you will need to create your own "removedAdsProduct" bool and save it in something like NSUserDefaults, Keychain or a class using NSCoding and than call this method when your app launches.

- To pause/resume tasks in your app/game when Ads are viewed you can implement the delegate methods if needed

Set the delegate in your GameScenes "didMoveToView" (init) method like so
```swift
Ads.sharedInstance.delegate = self
```

and create an extension conforming to the protocol (this helps with clean code as well)
```swift
extension GameScene: AdsDelegate {
    func pauseTasks() {
        // pause your game/app
    }
    func resumeTasks() { 
       // resume your game/app
    }
}
```

NOTE: There seems to a problem with AdMob Banner delegates not getting called. Therefore if you need your game/app to be paused you need call these 2 user methods in your AppDelegate.swift at the correct spots.

```swift
Ads.sharedInstance.adMobBannerClicked()
Ads.sharedInstance.adMobBannerClosed()
```

which ensures the AdsDelegate protocol gets called.

# When you go Live 

- Step 1: If you havent used iAds before make sure your account is set up for iAds. You mainly have to sign an agreement in your developer account. (https://developer.apple.com/iad/)

- Step 2: Sign up for a Google AdMob account and create your real ad IDs, 1 for banner and 1 for inter Ads. (https://support.google.com/admob/answer/2784575?hl=en-GB)

- Step 3: In Ads.swift in the struct called AdUnitId enter your real Ad IDs for both banner and inter ads.

- Step 4: When you submit your app on iTunes Connect do not forget to select YES for "Does your app use an advertising identifier", otherwise it will get rejected. If you decide to just use iAds than remove all google frameworks and references from your project and make sure you select NO, otherwise your app will also get rejected.

NOTE: - Dont forget to setup the "-D DEBUG" custom flag (step 1) or the helper will not work correctly with adMob.

# Resize view for banner ads

In SpriteKit games you normally dont want the view to resize when showing banner ads, however in UIKit apps this might be prefered. To resize your views you need to let apple create the Banners by themselves by using the canDisplayBannerAds bool. You should change the showBannerAd method so it looks like this

```swift
  func showBannerAd(...) {
        ...
        
        if iAdsAreSupported {
              presentingViewController.canDisplayBannerAds = true // uncomment line to resize view for banner ads
             //iAdLoadBannerAd() // comment out if canDisplayBanner ads is used because it now creates banner automatically
        } else {
            adMobLoadBannerAd() // not sure how to resize view with adMob banner
        }
    }
    }
```
NOTE:
This might not create a sharedBanner ad that can be used accross multiple viewControllers.

# Final Info

The sample project is the basic Apple spritekit template. It now shows a banner Ad on launch and an inter ad randomly when touching the screen. 
Like I mentioned above I primarly focused on SpriteKit to make it easy to call Ads from your SKScenes without having to use NSNotificationCenter or Delegates to constantly communicate with the viewController. Also this should help keep your viewController clean as mine became a mess after integrating AdMob.

Please let me know about any bugs or improvements, I am by now means an expert. 

Enjoy

# Release Notes

- v3.7.1

Improved the "removeBannerAd" method to ensure that all banner instances get removed incase there are multiple instances of a banner from the same AdNetwork.

Updated AdMob SDK to v 7.8.0

- v 3.7

Added a version without iAds. I will update the helper with another ad provider once I know 100% what is happening with iAds.

- v 3.6.2

Clean-Up and improvements

- v 3.6.1

Clean-Up and updated AdMob SDK to v7.7.1

- v 3.6

Updated to Swift 2.2

- v 3.5.2

Small fixes and improvements


- v 3.5.1

Updated Google AdMob SDK to v7.7.0

Note: It seems with this or one of the previous AdMob SDK updates it is possible to enable bitCode and not get a compiler error anymore. If you disabled bitCode due to the helper you should enable it again. 
Go to targets-BuildPhases and type bitCode into the search field and set it to yes. BitCode helps adjust your app size in the appStore so it should be turned on.

- v 3.5

Ads are now added to the apps rootViewController. This should ensure that ads are shown correctly when using multiple viewControllers.

You can now reverse the changes in "Not a SpriteKit game" as it is not needed anymore. The presentingViewController property is now only really used to get a reference to the rootViewController.

- v 3.4.1

Fixed a bug where interAds could get stuck in a loop trying to reload when having connectivity issues.

- v 3.4

Changed the way custom ads are handled. Please read the "How to use" section again.

- v 3.3 

Fixed a bug that could cause iAdBanners to get misaligned when viewed in portrait mode on iPads.

Added 2 new custom user methods "adMobBannerClicked" and "adMobBannerClosed" because the corresponding delegate methods do not work. You should call these, if needed, in your appDelegate at the correct spots.

Thanks you to member riklowe for pointing these out.

- v 3.2

iPadPro improvements

- v 3.1.2

Added the ability to use automatically created iAd banners which will resize your views

- v 3.1.1

Clean-up

- v 3.1

Removed iAd and AdMob banner properties from the appDelegate and moved them to Ads.swift because its a Singleton class and there is therefore only 1 instance of the banners anyway. 
If you used a previous version of this helper you can delete these in your "AppDelegate.swift".
```swift
let appDelegate...
var iAdBanner...
var adMobBanner...
```

- v 3.0

Added ability to show custom ads
Added extension to hide print statements for release
Small fixes, improvements and clean-up







 
