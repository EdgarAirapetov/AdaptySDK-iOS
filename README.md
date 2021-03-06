# Adapty iOS SDK

[![Version](https://img.shields.io/cocoapods/v/Adapty.svg?style=flat)](https://cocoapods.org/pods/Adapty)
[![License](https://img.shields.io/cocoapods/l/Adapty.svg?style=flat)](https://cocoapods.org/pods/Adapty)
[![Platform](https://img.shields.io/cocoapods/p/Adapty.svg?style=flat)](https://cocoapods.org/pods/Adapty)

![Adapty: Win back churned subscribers in your iOS app](https://raw.githubusercontent.com/adaptyteam/AdaptySDK-iOS/master/adapty.png)

Adapty helps you track business metrics, and lets you run ad campaigns targeted at churned users faster, written in Swift. https://adapty.io/

## Requirements

- iOS 12.0+
- Xcode 10.2+
- Swift 5+

## Installation

### CocoaPods

1. Create a Podfile if you don't have one: `pod init`
2. Add Adapty to your Podfile: `pod 'Adapty', '~> 1.1.0'`
3. Save the file and run: `pod install`. This creates an `.xcworkspace` file for your app. Use this file for all future development on your application.

## Usage

### Configure your app

In your AppDelegate class:

```Swift
import Adapty
```

And add the following to `application(_:didFinishLaunchingWithOptions:)`:

```Swift
Adapty.activate("YOUR_APP_KEY", customerUserId: "YOUR_USER_ID")
```

If your app doesn't have user IDs, you can use **`.activate("YOUR_APP_KEY")`** or pass nil for the **`customerUserId`**. Anyway, you can update **`customerUserId`** later within user update request.

### Observer Mode

In some cases, if you have already built a functioning subscription system, it may not be possible or feasible to use the Adapty SDK to make purchases. However, you can still use the SDK to get access to the data.

Just configure Adapty SDK in Observer Mode – update **`.activate`** method:

```Swift
Adapty.activate("YOUR_APP_KEY", customerUserId: "YOUR_USER_ID", observerMode: true)
```

### Update your user

Later you might want to update your user.

```Swift
Adapty.updateProfile(customerUserId: "<id-in-your-system>",
                     email: "example@email.com",
                     phoneNumber: "+1-###-###-####",
                     facebookUserId: "###############",
                     amplitudeUserId: "###",
                     mixpanelUserId: "###",
                     firstName: "Test",
                     lastName: "Test",
                     gender: "",
                     birthday: Date) { (error) in
                        if error == nil {
                            // successful update                              
                        }
}
```

All properties are optional.  
For **`gender`** possible values are: **`m`**, **`f`**, but you can also pass custom string value.

### Attribution tracker integration

To integrate with attribution system, just pass attribution you receive to Adapty method.

```Swift
Adapty.updateAttribution("<attribution>") { (error) in
    if error == nil {
        // successful update
    }
}
```

**`attribution`** is `Dictionary?` object.

Supported keys in **`attribution`** are the following:
```
network
campaign
trackerToken
trackerName
adgroup
creative
clickLabel
adid
```

To integrate with [AdjustSDK](https://github.com/adjust/ios_sdk), just pass attribution you receive from delegate method of Adjust iOS SDK `- (void)adjustAttributionChanged:(ADJAttribution *)attribution` to Adapty `updateAttribution` method.

### Get purchase containers (paywalls)

```Swift
Adapty.getPurchaseContainers { (containers, error) in
    // if error is empty, containers should contain info about your paywalls
}
```

### Make purchase

```Swift
Adapty.makePurchase(product: <product>, offerId: <offerId>) { (purchaserInfo, receipt, appleValidationResult, product, error) in
    if error == nil {
        // successful purchase
    }
    
    // response is a Dictionary, containing all info about receipt from AppStore
}
```

**`product`** is `ProductModel` object, it's required and can't be empty. You can get one from any available container. 
**`offerId`** is `String?` object, optional.
Adapty handles subscription offers signing for you as well.

### Restore purchases

```Swift
Adapty.restorePurchases { (error) in
    if error == nil {
        // successful restore
    }
}
```

### Validate your receipt

```Swift
Adapty.validateReceipt("<receiptEncoded>") { (response, error) in
    // response is a Dictionary, containing all info about receipt from AppStore
}
```

**`receiptEncoded`** is required and can't be empty.

### Get user purchases info

```Swift
Adapty.getPurchaserInfo { (purchaserInfo, error) in
    // purchaserInfo object contains all of the purchase and subscription data available about the user
}
```

The **`purchaserInfo`** object gives you access to the following information about a user:

| Name  | Description |
| -------- | ------------- |
| promotionalOfferEligibility | Boolean indicating whether the promotional offer is available for the customer. |
| introductoryOfferEligibility | Boolean indicating whether the introductory offer is available for the customer. |
| paidAccessLevels | Dictionary where the keys are paid access level identifiers configured by developer in Adapty dashboard. Values are PaidAccessLevelsInfoModel objects. Can be null if the customer has no access levels. |
| subscriptions | Dictionary where the keys are vendor product ids. Values are SubscriptionsInfoModel objects. Can be null if the customer has no subscriptions. |
| nonSubscriptions | Dictionary where the keys are vendor product ids. Values are array[] of NonSubscriptionsInfoModel objects. Can be null if the customer has no purchases. |
| appleValidationResult | Info received from Apple receipt validation. |

**`paidAccessLevels`** stores info about current users access level.

| Name  | Description |
| -------- | ------------- |
| id | Paid Access Level identifier configured by developer in Adapty dashboard. |
| isActive | Boolean indicating whether the paid access level is active. |
| vendorProductId | Identifier of the product in vendor system (App Store/Google Play etc.) that unlocked this access level. |
| store | The store that unlocked this subscription, can be one of: **app_store**, **play_store** & **adapty**. |
| activatedAt | The ISO 8601 datetime when access level was activated (may be in the future). |
| renewedAt | The ISO 8601 datetime when access level was renewed. |
| expiresAt | The ISO 8601 datetime when access level will expire (may be in the past and may be null for lifetime access). |
| isLifetime | Boolean indicating whether the paid access level is active for lifetime (no expiration date). If set to true you shouldn't use **expires_at**. |
| activeIntroductoryOfferType | The type of active introductory offer. Possible values are: **free_trial**, **pay_as_you_go** & **pay_up_front**. If the value is not null it means that offer was applied during the current subscription period. |
| activePromotionalOfferType | The type of active promotional offer. Possible values are: **free_trial**, **pay_as_you_go** & **pay_up_front**. If the value is not null it means that offer was applied during the current subscription period. |
| willRenew | Boolean indicating whether auto renewable subscription is set to renew. |
| isInGracePeriod | Boolean indicating whether auto renewable subscription is in grace period. |
| unsubscribedAt | The ISO 8601 datetime when auto renewable subscription was cancelled. Subscription can still be active, it just means that auto renewal turned off. Will set to null if the user reactivates subscription. |
| billingIssueDetectedAt | The ISO 8601 datetime when billing issue was detected (vendor was not able to charge the card). Subscription can still be active. Will set to null if the charge was successful. |

**`subscriptions`** stores info about vendor subscription.

| Name  | Description |
| -------- | ------------- |
| isActive | Boolean indicating whether the subscription is active. |
| vendorProductId | Identifier of the product in vendor system (App Store/Google Play etc.). |
| store | Store where the product was purchased. Possible values are: **app_store**, **play_store** & **adapty**. |
| activatedAt | The ISO 8601 datetime when access level was activated (may be in the future). |
| renewedAt | The ISO 8601 datetime when access level was renewed. |
| expiresAt | The ISO 8601 datetime when access level will expire (may be in the past and may be null for lifetime access). |
| isLifetime | Boolean indicating whether the subscription is active for lifetime (no expiration date). If set to true you shouldn't use **expires_at**. |
| activeIntroductoryOfferType | The type of active introductory offer. Possible values are: **free_trial**, **pay_as_you_go** & **pay_up_front**. If the value is not null it means that offer was applied during the current subscription period. |
| activePromotionalOfferType | The type of active promotional offer. Possible values are: **free_trial**, **pay_as_you_go** & **pay_up_front**. If the value is not null it means that offer was applied during the current subscription period. |
| willRenew | Boolean indicating whether auto renewable subscription is set to renew. |
| isInGracePeriod | Boolean indicating whether auto renewable subscription is in grace period. |
| unsubscribedAt | The ISO 8601 datetime when auto renewable subscription was cancelled. Subscription can still be active, it just means that auto renewal turned off. Will set to null if the user reactivates subscription. |
| billingIssueDetectedAt | The ISO 8601 datetime when billing issue was detected (vendor was not able to charge the card). Subscription can still be active. Will set to null if the charge was successful. |
| isSandbox | Boolean indicating whether the product was purchased in sandbox or production environment. |
| vendorTransactionId | Transaction id in vendor system. |
| vendorOriginalTransactionId | Original transaction id in vendor system. For auto renewable subscription this will be id of the first transaction in the subscription. |

**`nonSubscriptions `** stores info about purchases that are not subscriptions.

| Name  | Description |
| -------- | ------------- |
| purchaseId | Identifier of the purchase in Adapty. You can use it to unsure that you've already processed this purchase (for example tracking one time products). |
| vendorProductId | Identifier of the product in vendor system (App Store/Google Play etc.). |
| store | Store where the product was purchased. Possible values are: **app_store**, **play_store** & **adapty**. |
| purchasedAt | The ISO 8601 datetime when the product was purchased. |
| isOneTime | Boolean indicating whether the product should only be processed once. If true, the purchase will be returned by Adapty API one time only. |
| isSandbox | Boolean indicating whether the product was purchased in sandbox or production environment. |
| vendorTransactionId | Transaction id in vendor system. |
| vendorOriginalTransactionId | Original transaction id in vendor system. For auto renewable subscription this will be id of the first transaction in the subscription. |

### Checking if a user is subscribed 

The subscription status for a user can easily be determined from **`paidAccessLevels`** property of **`purchaserInfo`** object by **`isActive`** property inside.

```Swift
Adapty.getPurchaserInfo { (purchaserInfo, error) in
    if purchaserInfo?.paidAccessLevels["level_configured_in_dashboard"]?.isActive == true {
    
    }
}
```

### Method swizzling in Adapty

The Adapty SDK performs method swizzling for receiving your APNs token. Developers who prefer not to use swizzling can disable it by adding the flag AdaptyAppDelegateProxyEnabled in the app’s Info.plist file and setting it to NO (boolean value).

If you have disabled method swizzling, you'll need to explicitly send your APNs to Adapty. Override the methods didRegisterForRemoteNotificationsWithDeviceToken to retrieve the APNs token, and then set Adapty's apnsToken property:

```Swift
func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
    Adapty.apnsToken = deviceToken
}
```

## License

Adapty is available under the GNU license. [See LICENSE](https://github.com/adaptyteam/AdaptySDK-iOS/blob/master/LICENSE) for details.
