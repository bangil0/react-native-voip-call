
# react-native-voip-call

## Getting started
### Npm
```bash
$ npm install react-native-voip-call --save
```

### Yarn
```bash
$ yarn add react-native-voip-call
```

### Mostly automatic installation RN < 0.60.x

`$ react-native link react-native-voip-call`

## Usage

### 1. Background Push Kit Notification For Ios

##### 1.1 Configure Voip Service to App
 
 1. please Refer and Configure [Apple Voice Over IP](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/OptimizeVoIP.html)
 
 2. Make sure you enabled the folowing in `Xcode -> Signing & Capabilities:`

2.1) `Background Modes` -> `Voice over IP enabled` <br />
2.2 )`Capability` -> `Push Notifications`

3. Add Following Code to `Xcode -> project_folder -> AppDelegate.m`

```objective-c

...

#import "RNVoipCall.h"                          /* <------ add this line */  
#import <PushKit/PushKit.h>                    /* <------ add this line */
#import "RNVoipPushKit.h"                     /* <------ add this line */

@implementation AppDelegate

....

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    ....
}


// --- Handle updated push credentials
- (void)pushRegistry:(PKPushRegistry *)registry didUpdatePushCredentials:(PKPushCredentials *)credentials forType:(PKPushType)type {
  // Register VoIP push token (a property of PKPushCredentials) with server
  [RNVoipPushKit didUpdatePushCredentials:credentials forType:(NSString *)type];
}

- (void)pushRegistry:(PKPushRegistry *)registry didInvalidatePushTokenForType:(PKPushType)type
{
  // --- The system calls this method when a previously provided push token is no longer valid for use. No action is necessary on your part to reregister the push type. Instead, use this method to notify your server not to send push notifications using the matching push token.
}


// --- Handle incoming pushes (for ios <= 10)
- (void)pushRegistry:(PKPushRegistry *)registry didReceiveIncomingPushWithPayload:(PKPushPayload *)payload forType:(PKPushType)type {
   NSLog(@"Ajith");
  
  
  
  [RNVoipPushKit didReceiveIncomingPushWithPayload:payload forType:(NSString *)type];
}

// --- Handle incoming pushes (for ios >= 11)
- (void)pushRegistry:(PKPushRegistry *)registry didReceiveIncomingPushWithPayload:(PKPushPayload *)payload forType:(PKPushType)type withCompletionHandler:(void (^)(void))completion {
  
  
  
  NSString *callerName = @"RNVoip is Calling";
  NSString *callerId = [[[NSUUID UUID] UUIDString] lowercaseString];
  NSString *handle = @"1234567890";
  NSString *handleType = @"generic";
  BOOL hasVideo = false;
  
  
  @try {
    if([payload.dictionaryPayload[@"data"] isKindOfClass:[NSDictionary class]]){
      NSDictionary *dataPayload = payload.dictionaryPayload[@"data"];
      
      callerName = [dataPayload[@"name"] isKindOfClass:[NSString class]] ?  [NSString stringWithFormat: @"%@ is Calling", dataPayload[@"name"]] : @"RNVoip is Calling";
      
      callerId = [dataPayload[@"uuid"] isKindOfClass:[NSString class]] ?  dataPayload[@"uuid"] : [[[NSUUID UUID] UUIDString] lowercaseString];
      
      handle = [dataPayload[@"handle"] isKindOfClass:[NSString class]] ?  dataPayload[@"handle"] : @"1234567890";
      
      handleType = [dataPayload[@"handleType"] isKindOfClass:[NSString class]] ?  dataPayload[@"handleType"] : @"generic";
      
      hasVideo = dataPayload[@"hasVideo"] ? true : false;
      
    }
  } @catch (NSException *exception) {
    
    NSLog(@"Error PushKit payload %@", exception);
    
  } @finally {
    
    
    NSLog(@"RNVoip caller id ===> %@    callerNAme  ==> %@ handle  ==> %@",callerId, callerName, hasVideo ? @"true": @"false");
    
    NSDictionary *extra = [payload.dictionaryPayload valueForKeyPath:@"data"];
    
    [RNVoipCall reportNewIncomingCall:callerId handle:handle handleType:handleType hasVideo:hasVideo localizedCallerName:callerName fromPushKit: YES payload:extra withCompletionHandler:completion];
    
    [RNVoipPushKit didReceiveIncomingPushWithPayload:payload forType:(NSString *)type];
    
  }
}

@end

```
4. Add Following Code to `Xcode -> project_folder -> info.plist`

```xml
<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
    <string>voip</string>
    <string>fetch</string>
    <string>remote-notification</string>
</array>
```


5. `IosPushKitHandler.js`

```javascript
import React, {useEffect, useState } from 'react';
import {
  View,
  Text,
  Platform,
  Button
} from 'react-native';
import RNVoipCall, { RNVoipPushKit } from 'react-native-voip-call';

const IsIos = Platform.OS === 'ios';

const log = (data) => console.log('RNVoipCall===> ',data);

const IosPushKitHandler = () => { 
   const [pushkitToken, setPushkitToken] = useState(''); 

   useEffect(()=>{  
    iosPushKit(); 
  },[])
  
   const iosPushKit = () => {
    if(IsIos){
      //For Push Kit
      RNVoipPushKit.requestPermissions();  // --- optional, you can use another library to request permissions
      
       //Ios PushKit device token Listner
      RNVoipPushKit.getPushKitDeviceToken((res) => {
        if(res.platform === 'ios'){
            setPushkitToken(res.deviceToken)
        }
       });
       
       //On Remote Push notification Recived in Forground
       RNVoipPushKit.RemotePushKitNotificationReceived((notification)=>{
           log(notification);
       });
    }
  }
  
    return (
      <View>
        <Text>
          {"push kit token:" + pushkitToken}
        </Text>
      </View>
  );
};

}
export default IosPushKitHandler;

```

6. Create pem file for push please Refer [generate Cretificate](https://support.qiscus.com/hc/en-us/articles/360023340734-How-to-Create-Certificate-pem-for-Pushkit-) , [convert p12 to pem](https://stackoverflow.com/questions/40720524/how-to-send-push-notifications-to-test-ios-pushkit-integration-online)

7. send push [sendPush.php](https://github.com/ajith-ab/react-native-voip-call/blob/master/server/sendPush.php)
