# Quick Start Guide

This guide will help you get started with using the Apple APIs documented in this repository.

## Prerequisites

1. **Apple ID**: A valid Apple ID (free or paid developer account)
2. **Anisette Data**: Device-specific authentication data
3. **AltSign Library**: The AltSign library or equivalent implementation
4. **Development Environment**: macOS or Linux with appropriate build tools

## Obtaining Anisette Data

Anisette data is required for authentication. You should use an anisette v3 server for proper provisioning:

### Recommended: Anisette V3 Server

Use an official anisette v3 server for proper provisioning. For Swift projects, use the [RemoteAnisette](https://github.com/SideStore/RemoteAnisette) package.

### Server Options

- **Official OmniSette**: Use the official OmniSette v3 server
- **Anisette V3**: Use a trusted anisette v3 server
- **Self-Hosted**: Host your own anisette v3 server for maximum control

**Important**: Never manually create anisette data with hardcoded mid/otp values. Always use proper v3 server provisioning.

## Basic Authentication Flow

### Step 1: Create Anisette Data Object

```objective-c
ALTAnisetteData *anisetteData = [[ALTAnisetteData alloc] initWithMachineID:@"machine-id"
                                                            oneTimePassword:@"otp"
                                                                localUserID:@"local-user-id"
                                                                routingInfo:123456789
                                                     deviceUniqueIdentifier:@"device-uuid"
                                                         deviceSerialNumber:@"serial"
                                                          deviceDescription:@"iPhone12,1"
                                                                       date:[NSDate date]
                                                                     locale:[NSLocale localeWithLocaleIdentifier:@"en_US"]
                                                                   timeZone:[NSTimeZone timeZoneWithName:@"America/Los_Angeles"]];
```

### Step 2: Authenticate with Apple ID

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

[api authenticate:@"your@email.com"
          password:@"your-password"
      anisetteData:anisetteData
 verificationHandler:^(void (^completion)(NSString *)) {
     // Prompt user for 2FA code
     NSString *code = [self promptUserFor2FACode];
     completion(code);
 }
 completionHandler:^(ALTAccount *account, ALTAppleAPISession *session, NSError *error) {
    if (error) {
        NSLog(@"Authentication failed: %@", error);
        return;
    }
    
    NSLog(@"Authenticated successfully");
    NSLog(@"Account: %@", account.name);
    NSLog(@"DSID: %@", session.dsid);
    
    // Save session for subsequent requests
    self.currentSession = session;
    self.currentAccount = account;
}];
```

### Step 3: Fetch Teams

```objective-c
[api fetchTeamsForAccount:self.currentAccount 
                   session:self.currentSession 
          completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch teams: %@", error);
        return;
    }
    
    if (teams.count == 0) {
        NSLog(@"No teams available");
        return;
    }
    
    // Select the first team (or let user choose)
    self.currentTeam = teams.firstObject;
    NSLog(@"Selected team: %@", self.currentTeam.name);
    
    // Continue with next steps
    [self fetchDevices];
}];
```

### Step 4: Fetch Devices

```objective-c
- (void)fetchDevices {
    [[ALTAppleAPI sharedAPI] fetchDevicesForTeam:self.currentTeam 
                                           types:ALTDeviceTypeAll 
                                         session:self.currentSession 
                                completionHandler:^(NSArray<ALTDevice *> *devices, NSError *error) {
        if (error) {
            NSLog(@"Failed to fetch devices: %@", error);
            return;
        }
        
        NSLog(@"Found %lu devices", (unsigned long)devices.count);
        
        for (ALTDevice *device in devices) {
            NSLog(@"Device: %@ (UDID: %@)", device.name, device.identifier);
        }
        
        // Continue with next steps
        [self fetchCertificates];
    }];
}
```

### Step 5: Add Certificate

```objective-c
- (void)addCertificate {
    [[ALTAppleAPI sharedAPI] addCertificateWithMachineName:@"My Mac" 
                                                    toTeam:self.currentTeam 
                                                  session:self.currentSession 
                                       completionHandler:^(ALTCertificate *certificate, NSError *error) {
        if (error) {
            NSLog(@"Failed to add certificate: %@", error);
            return;
        }
        
        NSLog(@"Created certificate: %@", certificate.name);
        self.currentCertificate = certificate;
        
        // Save certificate as encrypted P12
        NSData *p12Data = [certificate encryptedP12DataWithPassword:@"secure-password"];
        [p12Data writeToFile:@"certificate.p12" atomically:YES];
        
        // Continue with next steps
        [self addAppID];
    }];
}
```

### Step 6: Add App ID

```objective-c
- (void)addAppID {
    [[ALTAppleAPI sharedAPI] addAppIDWithName:@"My App" 
                              bundleIdentifier:@"com.example.myapp" 
                                         team:self.currentTeam 
                                      session:self.currentSession 
                             completionHandler:^(ALTAppID *appID, NSError *error) {
        if (error) {
            NSLog(@"Failed to add App ID: %@", error);
            return;
        }
        
        NSLog(@"Created App ID: %@", appID.name);
        self.currentAppID = appID;
        
        // Update with entitlements
        [self updateAppID];
    }];
}
```

### Step 7: Update App ID with Entitlements

```objective-c
- (void)updateAppID {
    NSMutableDictionary *entitlements = [self.currentAppID.entitlements mutableCopy];
    entitlements[ALTEntitlementKeychainAccessGroups] = @[@"com.example.myapp.*"];
    
    // Check if paid-only entitlements can be used
    if (ALTFreeDeveloperCanUseEntitlement(ALTEntitlementIncreasedMemoryLimit)) {
        entitlements[ALTEntitlementIncreasedMemoryLimit] = @YES;
    }
    
    self.currentAppID.entitlements = entitlements;
    
    [[ALTAppleAPI sharedAPI] updateAppID:self.currentAppID 
                                    team:self.currentTeam 
                                 session:self.currentSession 
                        completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
        if (error) {
            NSLog(@"Failed to update App ID: %@", error);
            return;
        }
        
        NSLog(@"Updated App ID with entitlements");
        self.currentAppID = updatedAppID;
        
        // Continue with next steps
        [self fetchProvisioningProfile];
    }];
}
```

### Step 8: Fetch Provisioning Profile

```objective-c
- (void)fetchProvisioningProfile {
    [[ALTAppleAPI sharedAPI] fetchProvisioningProfileForAppID:self.currentAppID 
                                                    deviceType:ALTDeviceTypeiPhone 
                                                          team:self.currentTeam 
                                                       session:self.currentSession 
                                          completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
        if (error) {
            NSLog(@"Failed to fetch provisioning profile: %@", error);
            return;
        }
        
        NSLog(@"Fetched provisioning profile: %@", profile.name);
        NSLog(@"Expiration: %@", profile.expirationDate);
        
        // Save profile
        [profile.data writeToFile:@"profile.mobileprovision" atomically:YES];
        
        NSLog(@"Setup complete! You can now sign your app.");
    }];
}
```

## Complete Example

Here's a complete example that ties everything together:

```objective-c
#import <AltSign/AltSign.h>

@interface AppDelegate ()
@property (nonatomic, strong) ALTAccount *currentAccount;
@property (nonatomic, strong) ALTAppleAPISession *currentSession;
@property (nonatomic, strong) ALTTeam *currentTeam;
@property (nonatomic, strong) ALTCertificate *currentCertificate;
@property (nonatomic, strong) ALTAppID *currentAppID;
@end

@implementation AppDelegate

- (void)applicationDidFinishLaunching:(NSNotification *)aNotification {
    // Start the authentication and setup process
    [self authenticateAndSetup];
}

- (void)authenticateAndSetup {
    // Obtain anisette data from a v3 server
    // Use RemoteAnisette (Swift) or a similar v3 server client
    // For Objective-C, you may need to bridge from Swift or use a compatible v3 server client
    ALTAnisetteData *anisetteData = /* Obtain from v3 server provisioning */;
    
    // Authenticate
    ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
    [api authenticate:@"your@email.com"
              password:@"your-password"
          anisetteData:anisetteData
     verificationHandler:^(void (^completion)(NSString *)) {
         // Prompt user for 2FA code
         dispatch_async(dispatch_get_main_queue(), ^{
             NSAlert *alert = [[NSAlert alloc] init];
             alert.messageText = @"2FA Required";
             alert.informativeText = @"Enter the 2FA code sent to your device:";
             [alert addButtonWithTitle:@"Submit"];
             [alert addButtonWithTitle:@"Cancel"];
             
             NSTextField *input = [[NSTextField alloc] initWithFrame:NSMakeRect(0, 0, 200, 24)];
             [alert setAccessoryView:input];
             
            [alert beginSheetModalForWindow:nil completionHandler:^(NSModalResponse returnCode) {
                 if (returnCode == NSAlertFirstButtonReturn) {
                     completion(input.stringValue);
                 } else {
                     completion(nil);
                 }
             }];
         });
     }
     completionHandler:^(ALTAccount *account, ALTAppleAPISession *session, NSError *error) {
        if (error) {
            NSLog(@"Authentication failed: %@", error);
            return;
        }
        
        self.currentAccount = account;
        self.currentSession = session;
        
        NSLog(@"Authenticated successfully");
        [self fetchTeams];
    }];
}

- (void)fetchTeams {
    [[ALTAppleAPI sharedAPI] fetchTeamsForAccount:self.currentAccount 
                                           session:self.currentSession 
                                  completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
        if (error) {
            NSLog(@"Failed to fetch teams: %@", error);
            return;
        }
        
        if (teams.count == 0) {
            NSLog(@"No teams available");
            return;
        }
        
        self.currentTeam = teams.firstObject;
        NSLog(@"Selected team: %@", self.currentTeam.name);
        
        [self addCertificate];
    }];
}

- (void)addCertificate {
    [[ALTAppleAPI sharedAPI] addCertificateWithMachineName:@"My Mac" 
                                                    toTeam:self.currentTeam 
                                                  session:self.currentSession 
                                       completionHandler:^(ALTCertificate *certificate, NSError *error) {
        if (error) {
            NSLog(@"Failed to add certificate: %@", error);
            return;
        }
        
        NSLog(@"Created certificate: %@", certificate.name);
        self.currentCertificate = certificate;
        
        // Save certificate
        NSData *p12Data = [certificate encryptedP12DataWithPassword:@"secure-password"];
        [p12Data writeToFile:@"certificate.p12" atomically:YES];
        
        [self addAppID];
    }];
}

- (void)addAppID {
    [[ALTAppleAPI sharedAPI] addAppIDWithName:@"My App" 
                              bundleIdentifier:@"com.example.myapp" 
                                         team:self.currentTeam 
                                      session:self.currentSession 
                             completionHandler:^(ALTAppID *appID, NSError *error) {
        if (error) {
            NSLog(@"Failed to add App ID: %@", error);
            return;
        }
        
        NSLog(@"Created App ID: %@", appID.name);
        self.currentAppID = appID;
        
        [self updateAppID];
    }];
}

- (void)updateAppID {
    NSMutableDictionary *entitlements = [self.currentAppID.entitlements mutableCopy];
    entitlements[ALTEntitlementKeychainAccessGroups] = @[@"com.example.myapp.*"];
    
    self.currentAppID.entitlements = entitlements;
    
    [[ALTAppleAPI sharedAPI] updateAppID:self.currentAppID 
                                    team:self.currentTeam 
                                 session:self.currentSession 
                        completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
        if (error) {
            NSLog(@"Failed to update App ID: %@", error);
            return;
        }
        
        NSLog(@"Updated App ID with entitlements");
        self.currentAppID = updatedAppID;
        
        [self fetchProvisioningProfile];
    }];
}

- (void)fetchProvisioningProfile {
    [[ALTAppleAPI sharedAPI] fetchProvisioningProfileForAppID:self.currentAppID 
                                                    deviceType:ALTDeviceTypeiPhone 
                                                          team:self.currentTeam 
                                                       session:self.currentSession 
                                          completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
        if (error) {
            NSLog(@"Failed to fetch provisioning profile: %@", error);
            return;
        }
        
        NSLog(@"Fetched provisioning profile: %@", profile.name);
        NSLog(@"Expiration: %@", profile.expirationDate);
        
        [profile.data writeToFile:@"profile.mobileprovision" atomically:YES];
        
        NSLog(@"Setup complete!");
    }];
}

@end
```

## Common Workflows

### Registering a New Device

```objective-c
- (void)registerDevice:(NSString *)udid name:(NSString *)name {
    [[ALTAppleAPI sharedAPI] registerDeviceWithName:name 
                                         identifier:udid 
                                                type:ALTDeviceTypeiPhone 
                                               team:self.currentTeam 
                                            session:self.currentSession 
                               completionHandler:^(ALTDevice *device, NSError *error) {
        if (error) {
            if (error.code == ALTAppleAPIErrorDeviceAlreadyRegistered) {
                NSLog(@"Device already registered");
            } else {
                NSLog(@"Failed to register device: %@", error);
            }
        } else {
            NSLog(@"Registered device: %@", device.name);
        }
    }];
}
```

### Creating an App Group

```objective-c
- (void)createAppGroup:(NSString *)name identifier:(NSString *)identifier {
    [[ALTAppleAPI sharedAPI] addAppGroupWithName:name 
                                  groupIdentifier:identifier 
                                              team:self.currentTeam 
                                           session:self.currentSession 
                              completionHandler:^(ALTAppGroup *group, NSError *error) {
        if (error) {
            NSLog(@"Failed to add App Group: %@", error);
            return;
        }
        
        NSLog(@"Created App Group: %@", group.name);
        
        // Assign to App ID
        [[ALTAppleAPI sharedAPI] assignAppID:self.currentAppID 
                                  toGroups:@[group] 
                                      team:self.currentTeam 
                                   session:self.currentSession 
                          completionHandler:^(BOOL success, NSError *error) {
            if (error) {
                NSLog(@"Failed to assign App ID to group: %@", error);
            } else {
                NSLog(@"Assigned App ID to group");
            }
        }];
    }];
}
```

### Refreshing a Provisioning Profile

```objective-c
- (void)refreshProvisioningProfile {
    [[ALTAppleAPI sharedAPI] fetchProvisioningProfileForAppID:self.currentAppID 
                                                    deviceType:ALTDeviceTypeiPhone 
                                                          team:self.currentTeam 
                                                       session:self.currentSession 
                                          completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
        if (error) {
            NSLog(@"Failed to fetch provisioning profile: %@", error);
            return;
        }
        
        NSLog(@"Refreshed provisioning profile");
        NSLog(@"New expiration: %@", profile.expirationDate);
        
        [profile.data writeToFile:@"profile.mobileprovision" atomically:YES];
    }];
}
```

## Error Handling

### Common Errors and Solutions

#### Authentication Failed (-20101, -22406)
```
Error: Incorrect credentials
Solution: Verify Apple ID and password are correct
```

#### Invalid Anisette Data (-22421)
```
Error: Invalid anisette data
Solution: Ensure anisette data is current and valid
```

#### Device Already Registered (35)
```
Error: Device already registered
Solution: Device is already in your developer account
```

#### Maximum App ID Limit Reached (9120)
```
Error: Maximum App ID limit reached
Solution: Delete unused App IDs or upgrade to paid account
```

#### Bundle Identifier Unavailable (9401)
```
Error: Bundle identifier unavailable
Solution: Choose a different bundle identifier
```

### Error Handling Pattern

```objective-c
- (void)handleError:(NSError *)error {
    if (!error) return;
    
    switch (error.code) {
        case ALTAppleAPIErrorIncorrectCredentials:
            NSLog(@"Incorrect Apple ID or password");
            break;
            
        case ALTAppleAPIErrorInvalidAnisetteData:
            NSLog(@"Invalid anisette data");
            break;
            
        case ALTAppleAPIErrorRequiresTwoFactorAuthentication:
            NSLog(@"2FA required but no handler provided");
            break;
            
        case ALTAppleAPIErrorDeviceAlreadyRegistered:
            NSLog(@"Device already registered");
            break;
            
        case ALTAppleAPIErrorMaximumAppIDLimitReached:
            NSLog(@"Maximum App ID limit reached");
            break;
            
        default:
            NSLog(@"Unknown error: %@", error);
            break;
    }
}
```

## Tips and Best Practices

1. **Save Session**: Save the authentication session to avoid re-authenticating
2. **Handle 2FA**: Always provide a verification handler for 2FA
3. **Check Limits**: Be aware of device and App ID limits for your account type
4. **Monitor Expiration**: Check provisioning profile expiration dates
5. **Secure Storage**: Store certificates and profiles securely
6. **Error Handling**: Implement comprehensive error handling
7. **Rate Limiting**: Implement exponential backoff for failed requests

## Next Steps

- Read the [GSA Authentication API](GSA-Authentication-API.md) documentation
- Read the [Developer Services API](Developer-Services-API.md) documentation
- Read the [Entitlements and Capabilities](Entitlements-and-Capabilities.md) documentation
- Check the [Troubleshooting Guide](Troubleshooting-Guide.md) for common issues

## References

- [AltSign Repository](https://github.com/sidestore/AltSign)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
