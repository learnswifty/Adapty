# Implementation Guide: Integrating In-App Purchases Using Adapty SDK

This document outlines the step-by-step process for integrating in-app purchases using the Adapty SDK and AdaptyUI SDK in your iOS application.

---

## Key Points

- **UI/UX Updates:**
  - Only UI/UX changes to the custom in-app purchase screen for devices running iOS versions below 15.0, such as modifying table cells.
- **Logic Changes:**
  - Do not modify the purchase logic under any circumstances.
- **Issue Handling:**
  - Immediately contact the admin if any bugs or problems arise during the purchase or restore processes.
- **File Handling:**
  - Do not directly copy or drag-drop files or folders from in-app purchase code.
  - Create a new file in the existing project and then copy only the text or code content.
  
----  

## Prerequisites

Ensure the following dependencies are added to your Podfile:

```ruby
pod 'Adapty', '3.2.0'
pod 'AdaptyUI', '3.2.0'
pod 'AppsFlyerFramework'
```

Run the following command to install these pods:

```bash
pod install
```

---

## Step 1: Define Constants

Create a `Constant` class to manage all app-specific constants:

```swift
class Constant {
    struct AppInfo {
        // Add app-specific information if required
    }
    
    struct AppURL {
        // Add URL-related constants if required
    }
    
    struct AppConstants {
        static let adaptyApiKey                 = "" // Replace with your Adapty API key
        static let accessLevelId               = "premium"
        static let placementId                 = "premium_1"
        static let kIS_IN_APP_PURCHASE         = "IsActive"
        static let kUSER_IS_IN_TRIAL_PERIOD    = "kUSER_IS_IN_TRIAL_PERIOD"
    }
}

// ADAPTY

var RC_ADAPTY_ARRAY_PLACEMENT                               = "pk_premium_1"


var RC_ADAPTY_PAYWALL_AB                                    = 0

var RC_ADAPTY_SHOW_FREE_TRIAL_ONCLOSE                       = 1

var RC_ADAPTY_EXTRA_PLACEMENTID                             = "trial_1"

var RC_ADAPTY_PLACEMENTID                                   = ""

```

---

## Step 2: Configure SDKs

In `AppDelegate.swift`, create a function to set up the required SDKs:

```swift
private func setupSDKs() {
    PurchaseService.shared.setupIAP()
    AppUtility.lockOrientation(.portrait, andRotateTo: .portrait)
}
```

---

## Step 3: Initialize SDKs on Launch

Call the `setupSDKs()` method inside the `didFinishLaunchingWithOptions` function in `AppDelegate.swift`:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    setupUI()
    setupSDKs()
    configureSplashScreen()
    return true
}
```

---

## Step 4: Display Adapty Paywall or Custom In-App Purchase Screen

### Method to Present Paywall

Implement the following method to display the paywall or a custom in-app purchase screen:

```swift
@objc func getPremium() {
    Task { @MainActor in
        if Connectivity.isConnectedToInternet {
            SVProgressHUD.show()
            
            if #available(iOS 15.0, *) {
                Task {
                    await PurchaseService.shared.presentPaywallOn(showVC: self)
                }
            } else {
                // Only change UI/UX for versions below iOS 15.0
                let premiumVC = UpgradeVC(
                    onWillDismiss: {
                        print("UpgradeView Will Dismiss")
                    },
                    onDidDismissVC: {
                        print("UpgradeView Did Dismiss")
                    },
                    onPurchasedFailed: { error in
                        print("Purchase Error: \(String(describing: error?.localizedDescription))")
                        // Immediate contact with admin if issue occurs
                    },
                    onPurchaseComplete: {
                        print("Successfully Purchased.")
                        // Logic to remove ads or unlock premium features
                    },
                    onPurchaseRestored: {
                        print("Successfully Restored.")
                        // Logic to remove ads or unlock premium features
                    },
                    onRestoredFailed: { error in
                        print("Restore Error: \(String(describing: error?.localizedDescription))")
                        // Immediate contact with admin if issue occurs
                    }
                )
                
                premiumVC.modalTransitionStyle = .crossDissolve
                premiumVC.modalPresentationStyle = .fullScreen
                self.navigationController?.present(premiumVC, animated: true)
            }
        } else {
            NetworkFiles.alertNetworkNotConnected(self) { _ in }
        }
    }
}
```

---


