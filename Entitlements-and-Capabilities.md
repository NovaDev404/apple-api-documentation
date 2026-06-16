# Entitlements and Capabilities Reference

## Overview

Apple uses entitlements and capabilities to grant specific permissions and features to applications. This document provides a comprehensive reference for all entitlements, capabilities, and features used in AltSign.

## Entitlements

Entitlements are key-value pairs that grant specific permissions to an application. They are embedded in the application's code signature and provisioning profile.

### Core Entitlements

#### ALTEntitlementApplicationIdentifier

**Key:** `application-identifier`

**Description:** The bundle identifier of the application, prefixed with the team identifier.

**Format:** `<TEAM_IDENTIFIER>.<BUNDLE_IDENTIFIER>`

**Example:** `ABCDEFGHIJ.com.example.myapp`

**Availability:** All account types

**Usage:**
```xml
<key>application-identifier</key>
<string>ABCDEFGHIJ.com.example.myapp</string>
```

#### ALTEntitlementKeychainAccessGroups

**Key:** `keychain-access-groups`

**Description:** Array of keychain access groups the application can access.

**Format:** Array of strings in format `<TEAM_IDENTIFIER>.<BUNDLE_IDENTIFIER>` or `<TEAM_IDENTIFIER>.<BUNDLE_IDENTIFIER>.*`

**Example:**
```xml
<key>keychain-access-groups</key>
<array>
    <string>ABCDEFGHIJ.com.example.myapp</string>
    <string>ABCDEFGHIJ.com.example.myapp.*</string>
</array>
```

**Availability:** All account types

**Notes:** The wildcard `.*` allows access to all keychain items with the same bundle identifier prefix.

#### ALTEntitlementAppGroups

**Key:** `com.apple.security.application-groups`

**Description:** Array of App Group identifiers the application belongs to.

**Format:** Array of strings starting with `group.`

**Example:**
```xml
<key>com.apple.security.application-groups</key>
<array>
    <string>group.com.example.myapp</string>
</array>
```

**Availability:** All account types

**Notes:** App Groups enable data sharing between apps and app extensions.

#### ALTEntitlementGetTaskAllow

**Key:** `get-task-allow`

**Description:** Allows the application to be debugged by attaching a debugger.

**Format:** Boolean

**Example:**
```xml
<key>get-task-allow</key>
<true/>
```

**Availability:** Development only

**Notes:** This entitlement is automatically added to development provisioning profiles and should NOT be included in App Store builds.

#### ALTEntitlementIncreasedMemoryLimit

**Key:** `com.apple.developer.increased-memory-limit`

**Description:** Increases the memory limit for the application on supported devices.

**Format:** Boolean

**Example:**
```xml
<key>com.apple.developer.increased-memory-limit</key>
<true/>
```

**Availability:** Paid accounts only

**Notes:** 
- Not available for free developer accounts
- Requires device capability support
- Useful for memory-intensive applications

#### ALTEntitlementTeamIdentifier

**Key:** `com.apple.developer.team-identifier`

**Description:** The team identifier for the development team.

**Format:** 10-character alphanumeric string

**Example:**
```xml
<key>com.apple.developer.team-identifier</key>
<string>ABCDEFGHIJ</string>
```

**Availability:** All account types

**Notes:** Automatically added by Apple; should not be manually specified.

#### ALTEntitlementInterAppAudio

**Key:** `com.apple.developer.interapp-audio`

**Description:** Allows the application to use Inter-App Audio for audio communication between apps.

**Format:** Boolean

**Example:**
```xml
<key>com.apple.developer.interapp-audio</key>
<true/>
```

**Availability:** Paid accounts only

**Notes:** 
- Not available for free developer accounts
- Requires specific audio session configuration

#### ALTEntitlementIncreasedDebuggingMemoryLimit

**Key:** `com.apple.developer.increased-debugging-memory-limit`

**Description:** Increases the memory limit specifically for debugging.

**Format:** Boolean

**Example:**
```xml
<key>com.apple.developer.increased-debugging-memory-limit</key>
<true/>
```

**Availability:** Paid accounts only

**Notes:** 
- Not available for free developer accounts
- Only affects debugging builds

#### ALTEntitlementExtendedVirtualAddressing

**Key:** `com.apple.developer.extended-virtual-addressing`

**Description:** Enables extended virtual addressing for large memory applications.

**Format:** Boolean

**Example:**
```xml
<key>com.apple.developer.extended-virtual-addressing</key>
<true/>
```

**Availability:** Paid accounts only

**Notes:** 
- Not available for free developer accounts
- Requires 64-bit architecture
- Useful for applications with large memory requirements

### Additional Common Entitlements

#### Network Extensions

**Key:** `com.apple.developer.networking.networkextension`

**Description:** Allows the application to use Network Extensions.

**Availability:** Paid accounts only, requires special approval

#### Associated Domains

**Key:** `com.apple.developer.associated-domains`

**Description:** Allows the application to associate with websites for universal links and shared credentials.

**Format:** Array of domain patterns

**Example:**
```xml
<key>com.apple.developer.associated-domains</key>
<array>
    <string>webcredentials:example.com</string>
    <string>applinks:example.com</string>
</array>
```

**Availability:** All account types

#### Push Notifications

**Key:** `aps-environment`

**Description:** Enables Apple Push Notification service.

**Format:** String (development or production)

**Example:**
```xml
<key>aps-environment</key>
<string>development</string>
```

**Availability:** All account types

#### iCloud

**Key:** `com.apple.developer.icloud-container-identifiers`

**Description:** Enables iCloud container access.

**Format:** Array of container identifiers

**Example:**
```xml
<key>com.apple.developer.icloud-container-identifiers</key>
<array>
    <string>iCloud.com.example.myapp</string>
</array>
```

**Availability:** Paid accounts only

#### Siri

**Key:** `com.apple.developer.siri`

**Description:** Enables Siri integration.

**Availability:** Paid accounts only, requires special approval

#### Apple Pay

**Key:** `com.apple.developer.pass-type-identifiers`

**Description:** Enables Apple Pay integration.

**Format:** Array of pass type identifiers

**Example:**
```xml
<key>com.apple.developer.pass-type-identifiers</key>
<array>
    <string>com.example.pass</string>
</array>
```

**Availability:** Paid accounts only

## Capabilities

Capabilities are high-level features that can be enabled for App IDs. They often map to one or more entitlements.

### ALTCapabilityIncreasedMemoryLimit

**Description:** Increases the memory limit for the application.

**Maps to:** `ALTEntitlementIncreasedMemoryLimit`

**Availability:** Paid accounts only

**Usage in AltSign:**
```objective-c
appID.capabilities = @{
    ALTCapabilityIncreasedMemoryLimit: @YES
};
```

### ALTCapabilityIncreasedDebuggingMemoryLimit

**Description:** Increases the memory limit specifically for debugging.

**Maps to:** `ALTEntitlementIncreasedDebuggingMemoryLimit`

**Availability:** Paid accounts only

**Usage in AltSign:**
```objective-c
appID.capabilities = @{
    ALTCapabilityIncreasedDebuggingMemoryLimit: @YES
};
```

### ALTCapabilityExtendedVirtualAddressing

**Description:** Enables extended virtual addressing for large memory applications.

**Maps to:** `ALTEntitlementExtendedVirtualAddressing`

**Availability:** Paid accounts only

**Usage in AltSign:**
```objective-c
appID.capabilities = @{
    ALTCapabilityExtendedVirtualAddressing: @YES
};
```

## Features

Features are user-facing capabilities that map to entitlements.

### ALTFeatureGameCenter

**Description:** Enables Game Center integration.

**Maps to:** Various Game Center entitlements

**Availability:** All account types

**Usage in AltSign:**
```objective-c
appID.features = @{
    ALTFeatureGameCenter: @YES
};
```

### ALTFeatureAppGroups

**Description:** Enables App Groups for data sharing between apps.

**Maps to:** `ALTEntitlementAppGroups`

**Availability:** All account types

**Usage in AltSign:**
```objective-c
appID.features = @{
    ALTFeatureAppGroups: @YES
};
```

### ALTFeatureInterAppAudio

**Description:** Enables Inter-App Audio for audio communication.

**Maps to:** `ALTEntitlementInterAppAudio`

**Availability:** Paid accounts only

**Usage in AltSign:**
```objective-c
appID.features = @{
    ALTFeatureInterAppAudio: @YES
};
```

## Free Developer Account Restrictions

Free developer accounts have restrictions on certain entitlements. Use `ALTFreeDeveloperCanUseEntitlement()` to check if an entitlement is available.

### Allowed Entitlements (Free Accounts)

- `ALTEntitlementApplicationIdentifier` - Always allowed
- `ALTEntitlementKeychainAccessGroups` - Always allowed
- `ALTEntitlementAppGroups` - Always allowed
- `ALTEntitlementGetTaskAllow` - Development only
- `ALTEntitlementTeamIdentifier` - Always allowed
- `ALTEntitlementAssociatedDomains` - Always allowed
- `ALTEntitlementAPSEnvironment` - Always allowed

### Restricted Entitlements (Paid Only)

- `ALTEntitlementIncreasedMemoryLimit` - Paid only
- `ALTEntitlementInterAppAudio` - Paid only
- `ALTEntitlementIncreasedDebuggingMemoryLimit` - Paid only
- `ALTEntitlementExtendedVirtualAddressing` - Paid only
- `ALTEntitlementNetworkExtensions` - Paid only, special approval
- `ALTEntitlementICloud` - Paid only
- `ALTEntitlementSiri` - Paid only, special approval
- `ALTEntitlementApplePay` - Paid only

### Checking Entitlement Availability

```objective-c
ALTEntitlement entitlement = ALTEntitlementIncreasedMemoryLimit;
if (ALTFreeDeveloperCanUseEntitlement(entitlement)) {
    // Entitlement is available for free accounts
    appID.entitlements[entitlement] = @YES;
} else {
    // Entitlement requires paid account
    NSLog(@"This entitlement requires a paid developer account");
}
```

## Entitlement to Feature Mapping

### ALTEntitlementForFeature

Converts a feature to its corresponding entitlement.

```objective-c
ALTFeature feature = ALTFeatureAppGroups;
ALTEntitlement entitlement = ALTEntitlementForFeature(feature);
// Returns: ALTEntitlementAppGroups
```

### ALTFeatureForEntitlement

Converts an entitlement to its corresponding feature.

```objective-c
ALTEntitlement entitlement = ALTEntitlementAppGroups;
ALTFeature feature = ALTFeatureForEntitlement(entitlement);
// Returns: ALTFeatureAppGroups
```

## Complete Entitlements File Example

### Development Entitlements (Free Account)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>application-identifier</key>
    <string>ABCDEFGHIJ.com.example.myapp</string>
    <key>keychain-access-groups</key>
    <array>
        <string>ABCDEFGHIJ.com.example.myapp.*</string>
    </array>
    <key>com.apple.developer.team-identifier</key>
    <string>ABCDEFGHIJ</string>
    <key>get-task-allow</key>
    <true/>
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.example.myapp</string>
    </array>
</dict>
</plist>
```

### Production Entitlements (Paid Account)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>application-identifier</key>
    <string>ABCDEFGHIJ.com.example.myapp</string>
    <key>keychain-access-groups</key>
    <array>
        <string>ABCDEFGHIJ.com.example.myapp.*</string>
    </array>
    <key>com.apple.developer.team-identifier</key>
    <string>ABCDEFGHIJ</string>
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.example.myapp</string>
    </array>
    <key>com.apple.developer.increased-memory-limit</key>
    <true/>
    <key>com.apple.developer.extended-virtual-addressing</key>
    <true/>
    <key>aps-environment</key>
    <string>production</string>
</dict>
</plist>
```

## Usage in AltSign

### Setting Entitlements

```objective-c
// Create App ID
[api addAppIDWithName:@"My App" 
       bundleIdentifier:@"com.example.myapp" 
                  team:team 
               session:session 
      completionHandler:^(ALTAppID *appID, NSError *error) {
    if (error) {
        NSLog(@"Failed to add App ID: %@", error);
        return;
    }
    
    // Set entitlements
    NSMutableDictionary *entitlements = [appID.entitlements mutableCopy];
    entitlements[ALTEntitlementKeychainAccessGroups] = @[@"com.example.myapp.*"];
    entitlements[ALTEntitlementAppGroups] = @[@"group.com.example.myapp"];
    
    // Check if paid-only entitlements can be used
    if (ALTFreeDeveloperCanUseEntitlement(ALTEntitlementIncreasedMemoryLimit)) {
        entitlements[ALTEntitlementIncreasedMemoryLimit] = @YES;
    }
    
    appID.entitlements = entitlements;
    
    // Update App ID
    [api updateAppID:appID 
                team:team 
             session:session 
    completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
        if (error) {
            NSLog(@"Failed to update App ID: %@", error);
        } else {
            NSLog(@"Updated App ID with entitlements");
        }
    }];
}];
```

### Setting Features

```objective-c
// Set features
NSMutableDictionary *features = [appID.features mutableCopy];
features[ALTFeatureAppGroups] = @YES;
features[ALTFeatureGameCenter] = @YES;

appID.features = features;

// Update App ID
[api updateAppID:appID 
            team:team 
         session:session 
completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
    // Handle response
}];
```

### Setting Capabilities

```objective-c
// Set capabilities
NSMutableDictionary *capabilities = [appID.capabilities mutableCopy];
capabilities[ALTCapabilityIncreasedMemoryLimit] = @YES;

appID.capabilities = capabilities;

// Update App ID
[api updateAppID:appID 
            team:team 
         session:session 
completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
    // Handle response
}];
```

## Entitlements in Provisioning Profiles

Provisioning profiles contain the entitlements that are actually granted to the application. The profile's entitlements must match or be a subset of the App ID's entitlements.

### Reading Entitlements from Profile

```objective-c
ALTProvisioningProfile *profile = [[ALTProvisioningProfile alloc] initWithData:profileData];
NSDictionary *entitlements = profile.entitlements;

NSLog(@"Entitlements: %@", entitlements);
```

### Common Profile Entitlements

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>ABCDEFGHIJ.com.example.myapp.*</string>
    </array>
    <key>application-identifier</key>
    <string>ABCDEFGHIJ.com.example.myapp</string>
    <key>com.apple.developer.team-identifier</key>
    <string>ABCDEFGHIJ</string>
    <key>get-task-allow</key>
    <true/>
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.example.myapp</string>
    </array>
    <key>provisioning-profile-id</key>
    <string>XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX</string>
</dict>
</plist>
```

## Troubleshooting

### Entitlement Mismatch

**Error:** "Code signing error: entitlements mismatch"

**Cause:** The entitlements in the app's embedded entitlements don't match the provisioning profile.

**Solution:** Ensure the app's entitlements match or are a subset of the profile's entitlements.

### Invalid Entitlement

**Error:** "The entitlements in your app bundle signature do not match the ones that are contained in the provisioning profile."

**Cause:** Attempting to use an entitlement not allowed by the account type or not in the profile.

**Solution:** Remove the entitlement or upgrade to a paid account if required.

### Missing Entitlement

**Error:** "This app is not allowed to use [feature]"

**Cause:** Required entitlement is missing from the profile.

**Solution:** Add the entitlement to the App ID and regenerate the provisioning profile.

## Best Practices

1. **Minimum Privilege:** Only request entitlements that your app actually needs
2. **Account Type Awareness:** Be aware of which entitlements require paid accounts
3. **Profile Management:** Keep provisioning profiles up to date with entitlement changes
4. **Testing:** Test with both development and production profiles
5. **Documentation:** Document which entitlements your app requires and why
6. **Validation:** Validate entitlements before building and signing

## References

- [Apple Developer - Entitlements](https://developer.apple.com/documentation/bundleresources/entitlements)
- [Apple Developer - Capabilities](https://developer.apple.com/documentation/xcode/enabling-app-capabilities)
- [AltSign Repository](https://github.com/sidestore/AltSign)
