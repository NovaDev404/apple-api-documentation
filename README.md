# Apple APIs Documentation for AltSign

This documentation provides comprehensive information about all Apple APIs used in the AltSign project, including their endpoints, authentication methods, request/response formats, and usage examples.

## Table of Contents

1. [GSA (Apple's Authentication Service)](#gsa-apples-authentication-service)
2. [Developer Services API](#developer-services-api)
   - [Teams API](#teams-api)
   - [Devices API](#devices-api)
   - [Certificates API](#certificates-api)
   - [App IDs API](#app-ids-api)
   - [App Groups API](#app-groups-api)
   - [Provisioning Profiles API](#provisioning-profiles-api)

## Overview

AltSign interacts with several Apple APIs to enable sideloading of iOS applications without an Apple Developer Program membership. These APIs are primarily used for:

- Authenticating with Apple ID (including 2FA)
- Managing developer teams
- Registering devices
- Creating and managing certificates
- Creating and managing App IDs
- Creating and managing App Groups
- Generating provisioning profiles

---

## GSA (Apple's Authentication Service)

The GSA (Apple's Authentication Service) is used to authenticate users with their Apple ID and handle two-factor authentication.

### Base URL
```
https://gsa.apple.com
```

### Authentication Flow

The authentication process uses SRP (Secure Remote Password) protocol with the following steps:

1. **Initial Handshake** - Client sends public key and receives server's public key, salt, and iteration count
2. **Verification** - Client sends verification message (M1) and receives server's verification message (M2)
3. **2FA Handling** - If required, handles SMS or trusted device verification
4. **Token Fetching** - Obtains authentication token for Developer Services API

### Endpoints

#### 1. Authentication Endpoint
```
POST https://gsa.apple.com/grandslam/GsService2
```

**Headers:**
```
Content-Type: text/x-xml-plist
X-MMe-Client-Info: <device description>
Accept: */*
User-Agent: akd/1.0 CFNetwork/978.0.7 Darwin/18.7.0
```

**Request Body (Property List):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Header</key>
    <dict>
        <key>Version</key>
        <string>1.0.1</string>
    </dict>
    <key>Request</key>
    <dict>
        <key>A2k</key>
        <data><client public key></data>
        <key>cpd</key>
        <dict>
            <key>bootstrap</key>
            <true/>
            <key>icscrec</key>
            <true/>
            <key>pbe</key>
            <false/>
            <key>prkgen</key>
            <true/>
            <key>svct</key>
            <string>iCloud</string>
            <key>loc</key>
            <string>en_US</string>
            <key>X-Apple-Locale</key>
            <string>en_US</string>
            <key>X-Apple-I-MD</key>
            <string><one-time password></string>
            <key>X-Apple-I-MD-M</key>
            <string><machine ID></string>
            <key>X-Mme-Device-Id</key>
            <string><device unique identifier></string>
            <key>X-Apple-I-MD-LU</key>
            <string><local user ID></string>
            <key>X-Apple-I-MD-RINFO</key>
            <integer><routing info></integer>
            <key>X-Apple-I-SRL-NO</key>
            <string><device serial number></string>
            <key>X-Apple-I-Client-Time</key>
            <string><current time></string>
            <key>X-Apple-I-TimeZone</key>
            <string>PST</string>
        </dict>
        <key>ps</key>
        <array>
            <string>s2k</string>
            <string>s2k_fo</string>
        </array>
        <key>o</key>
        <string>init</string>
        <key>u</key>
        <string><apple id></string>
    </dict>
</dict>
</plist>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Response</key>
    <dict>
        <key>c</key>
        <string><c value></string>
        <key>s</key>
        <data><salt></data>
        <key>i</key>
        <integer><iterations></integer>
        <key>B</key>
        <data><server public key></data>
        <key>sp</key>
        <string>s2k_fo</string>
        <key>Status</key>
        <dict>
            <key>ec</key>
            <integer>0</integer>
        </dict>
    </dict>
</dict>
</plist>
```

#### 2. Trusted Device 2FA Request
```
POST https://gsa.apple.com/auth/verify/trusteddevice
```

**Headers:**
```
Accept: application/x-buddyml
Accept-Language: en-us
Content-Type: application/x-plist
User-Agent: Xcode
X-Apple-App-Info: com.apple.gs.xcode.auth
X-Xcode-Version: 11.2 (11B41)
X-Apple-Identity-Token: <base64 encoded dsid:idmsToken>
X-Apple-I-MD-M: <machine ID>
X-Apple-I-MD: <one-time password>
X-Apple-I-MD-LU: <local user ID>
X-Apple-I-MD-RINFO: <routing info>
X-Mme-Device-Id: <device unique identifier>
X-MMe-Client-Info: <device description>
X-Apple-I-Client-Time: <current time>
X-Apple-Locale: <locale>
X-Apple-I-TimeZone: <timezone>
```

#### 3. Trusted Device 2FA Verify
```
POST https://gsa.apple.com/grandslam/GsService2/validate
```

**Headers:** Same as above, plus:
```
security-code: <verification code>
```

#### 4. SMS 2FA Request
```
POST https://gsa.apple.com/auth/verify/phone/put?mode=sms
```

**Request Body (Property List):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>serverInfo</key>
    <dict>
        <key>phoneNumber.id</key>
        <string>1</string>
    </dict>
</dict>
</plist>
```

#### 5. SMS 2FA Verify
```
POST https://gsa.apple.com/auth/verify/phone/securitycode?referrer=/auth/verify/phone/put
```

**Request Body (Property List):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>securityCode.code</key>
    <string><verification code></string>
    <key>serverInfo</key>
    <dict>
        <key>mode</key>
        <string>sms</string>
        <key>phoneNumber.id</key>
        <string>1</string>
    </dict>
</dict>
</plist>
```

### Anisette Data

Anisette data is required for all GSA requests and contains device-specific information:

- **machineID**: Device machine identifier
- **oneTimePassword**: One-time password for authentication
- **localUserID**: Local user identifier
- **routingInfo**: Routing information (unsigned long long)
- **deviceUniqueIdentifier**: Unique device identifier
- **deviceSerialNumber**: Device serial number
- **deviceDescription**: Device description string
- **date**: Current date
- **locale**: Device locale
- **timeZone**: Device timezone

### Error Codes

- `-20101`: Incorrect credentials
- `-22406`: Incorrect credentials
- `-22421`: Invalid anisette data
- `-21669`: Incorrect verification code (2FA)

### Usage Example (Swift)

```swift
import Foundation
import RemoteAnisette

// Obtain anisette data from a v3 server using RemoteAnisette
// https://github.com/SideStore/RemoteAnisette
let anisetteUser = try await AnisetteUser.provision(serial: "ani.sidestore.io")
let headers = try await anisetteUser.fetchHeaders()

// Convert headers to ALTAnisetteData
let anisetteData = ALTAnisetteData(
    machineID: headers["machineID"] ?? "",
    oneTimePassword: headers["oneTimePassword"] ?? "",
    localUserID: headers["localUserID"] ?? "",
    routingInfo: UInt64(headers["routingInfo"] ?? "0") ?? 0,
    deviceUniqueIdentifier: headers["deviceUniqueIdentifier"] ?? "",
    deviceSerialNumber: headers["deviceSerialNumber"] ?? "",
    deviceDescription: headers["deviceDescription"] ?? "",
    date: Date(),
    locale: Locale.current,
    timeZone: TimeZone.current
)

// Authenticate
let api = ALTAppleAPI.sharedAPI
api.authenticate(
    appleID: "your@email.com",
    password: "your-password",
    anisetteData: anisetteData,
    verificationHandler: { completion in
        // Prompt user for 2FA code
        let code = getUserInput()
        completion(code)
    }
) { account, session, error in
    if let error = error {
        print("Authentication failed: \(error)")
    } else {
        print("Authenticated successfully")
        print("Account: \(account)")
        print("Session: \(session)")
    }
}
```

---

## Developer Services API

The Developer Services API is used to manage Apple Developer resources including teams, devices, certificates, App IDs, App Groups, and provisioning profiles.

### Base URLs
```
https://developerservices2.apple.com/services/QH65B2/
https://developerservices2.apple.com/services/v1/
```

### Common Headers
```
Content-Type: text/x-xml-plist
User-Agent: Xcode
Accept: text/x-xml-plist
Accept-Language: en-us
X-Apple-App-Info: com.apple.gs.xcode.auth
X-Xcode-Version: 11.2 (11B41)
X-Apple-I-Identity-Id: <dsid>
X-Apple-GS-Token: <auth token>
X-Apple-I-MD-M: <machine ID>
X-Apple-I-MD: <one-time password>
X-Apple-I-MD-LU: <local user ID>
X-Apple-I-MD-RINFO: <routing info>
X-Mme-Device-Id: <device unique identifier>
X-MMe-Client-Info: <device description>
X-Apple-I-Client-Time: <current time>
X-Apple-Locale: <locale>
X-Apple-I-Locale: <locale>
X-Apple-I-TimeZone: <timezone>
```

### Common Request Parameters
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID (optional)></string>
    <!-- Additional parameters specific to endpoint -->
</dict>
</plist>
```

---

## Teams API

### Fetch Teams

Retrieves all teams associated with the authenticated account.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/listTeams.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>teams</key>
    <array>
        <dict>
            <key>teamId</key>
            <string><team ID></string>
            <key>name</key>
            <string><team name></string>
            <key>type</key>
            <string><team type></string>
        </dict>
    </array>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Team Types:**
- `Individual`: Personal team
- `Organization`: Organization team
- `Free`: Free developer team

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api fetchTeamsForAccount:account 
                   session:session 
          completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch teams: %@", error);
    } else {
        for (ALTTeam *team in teams) {
            NSLog(@"Team: %@ (ID: %@, Type: %ld)", team.name, team.identifier, (long)team.type);
        }
    }
}];
```

---

## Devices API

### Fetch Devices

Retrieves all registered devices for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listDevices.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>devices</key>
    <array>
        <dict>
            <key>deviceNumber</key>
            <string><device UDID></string>
            <key>name</key>
            <string><device name></string>
            <key>deviceClass</key>
            <string><device class></string>
            <key>model</key>
            <string><device model></string>
            <key>osVersion</key>
            <string><iOS version></string>
        </dict>
    </array>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

### Register Device

Registers a new device with the team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/addDevice.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>deviceNumber</key>
    <string><device UDID></string>
    <key>name</key>
    <string><device name></string>
    <key>DTDK_Platform</key>
    <string>ios</string>
</dict>
```

**Device Platforms:**
- `ios`: iPhone/iPad
- `tvos`: Apple TV

**Error Codes:**
- `35`: Device already registered or invalid device ID

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api registerDeviceWithName:@"My iPhone" 
                 identifier:@"<device UDID>" 
                        type:ALTDeviceTypeiPhone 
                       team:team 
                    session:session 
          completionHandler:^(ALTDevice *device, NSError *error) {
    if (error) {
        NSLog(@"Failed to register device: %@", error);
    } else {
        NSLog(@"Registered device: %@", device.name);
    }
}];
```

---

## Certificates API

### Fetch Certificates

Retrieves all development certificates for a team.

**Endpoint:**
```
GET https://developerservices2.apple.com/services/v1/certificates?filter[certificateType]=IOS_DEVELOPMENT
```

**Headers:** Same as common headers, plus team ID in headers.

**Response:**
```json
{
    "data": [
        {
            "id": "<certificate ID>",
            "type": "certificates",
            "attributes": {
                "certificateType": "IOS_DEVELOPMENT",
                "name": "<certificate name>",
                "serialNumber": "<serial number>",
                "certificateContent": "<base64 encoded certificate>",
                "expirationDate": "<ISO 8601 date>"
            }
        }
    ]
}
```

### Add Certificate

Creates a new development certificate by submitting a CSR (Certificate Signing Request).

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/submitDevelopmentCSR.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>csrContent</key>
    <string><base64 encoded CSR></string>
    <key>machineId</key>
    <string><machine UUID></string>
    <key>machineName</key>
    <string><machine name></string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>certRequest</key>
    <dict>
        <key>certificate</key>
        <data><base64 encoded certificate></data>
        <key>privateKey</key>
        <data><base64 encoded private key></data>
    </dict>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Error Codes:**
- `3250`: Invalid certificate request

### Revoke Certificate

Revokes an existing certificate.

**Endpoint:**
```
DELETE https://developerservices2.apple.com/services/v1/certificates/<certificate ID>
```

**Error Codes:**
- `7252`: Certificate does not exist

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api addCertificateWithMachineName:@"My Mac" 
                              toTeam:team 
                            session:session 
                 completionHandler:^(ALTCertificate *certificate, NSError *error) {
    if (error) {
        NSLog(@"Failed to add certificate: %@", error);
    } else {
        NSLog(@"Created certificate: %@", certificate.name);
        // Save certificate and private key
        NSData *p12Data = [certificate encryptedP12DataWithPassword:@"password"];
        [p12Data writeToFile:@"certificate.p12" atomically:YES];
    }
}];
```

---

## App IDs API

### Fetch App IDs

Retrieves all App IDs for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listAppIds.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>appIds</key>
    <array>
        <dict>
            <key>appIdId</key>
            <string><App ID ID></string>
            <key>name</key>
            <string><App ID name></string>
            <key>identifier</key>
            <string><bundle identifier></string>
            <key>entitlements</key>
            <dict>
                <key>keychain-access-groups</key>
                <array>
                    <string><bundle identifier>.*</string>
                </array>
            </dict>
        </dict>
    </array>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

### Add App ID

Creates a new App ID.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/addAppId.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>identifier</key>
    <string><bundle identifier></string>
    <key>name</key>
    <string><App ID name></string>
</dict>
```

**Error Codes:**
- `35`: Invalid App ID name
- `9120`: Maximum App ID limit reached
- `9401`: Bundle identifier unavailable
- `9412`: Invalid bundle identifier

### Update App ID

Updates an existing App ID with features and entitlements.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/updateAppId.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
    <key>entitlements</key>
    <dict>
        <key>application-identifier</key>
        <string><bundle identifier></string>
        <key>keychain-access-groups</key>
        <array>
            <string><bundle identifier>.*</string>
        </array>
    </dict>
</dict>
```

**Error Codes:**
- `35`: Invalid App ID name
- `9100`: App ID does not exist
- `9412`: Invalid bundle identifier

### Delete App ID

Deletes an existing App ID.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/deleteAppId.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
</dict>
```

**Error Codes:**
- `9100`: App ID does not exist

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api addAppIDWithName:@"My App" 
       bundleIdentifier:@"com.example.myapp" 
                  team:team 
               session:session 
      completionHandler:^(ALTAppID *appID, NSError *error) {
    if (error) {
        NSLog(@"Failed to add App ID: %@", error);
    } else {
        NSLog(@"Created App ID: %@", appID.name);
        
        // Update with entitlements
        appID.entitlements = @{
            ALTEntitlementKeychainAccessGroups: @[@"com.example.myapp.*"]
        };
        
        [api updateAppID:appID 
                    team:team 
                 session:session 
        completionHandler:^(ALTAppID *updatedAppID, NSError *error) {
            if (error) {
                NSLog(@"Failed to update App ID: %@", error);
            } else {
                NSLog(@"Updated App ID");
            }
        }];
    }
}];
```

---

## App Groups API

### Fetch App Groups

Retrieves all App Groups for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listApplicationGroups.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>applicationGroupList</key>
    <array>
        <dict>
            <key>applicationGroup</key>
            <dict>
                <key>applicationGroupIdentifier</key>
                <string><group identifier></string>
                <key>name</key>
                <string><group name></string>
                <key>applicationGroupId</key>
                <string><group ID></string>
            </dict>
        </dict>
    </array>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

### Add App Group

Creates a new App Group.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/addApplicationGroup.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>identifier</key>
    <string><group identifier></string>
    <key>name</key>
    <string><group name></string>
</dict>
```

**Error Codes:**
- `35`: Invalid App Group

### Assign App ID to Groups

Assigns an App ID to one or more App Groups.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/assignApplicationGroupToAppId.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
    <key>applicationGroups</key>
    <array>
        <string><group ID></string>
    </array>
</dict>
```

**Error Codes:**
- `9115`: App ID does not exist
- `35`: App Group does not exist

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api addAppGroupWithName:@"My App Group" 
          groupIdentifier:@"group.com.example.myapp" 
                      team:team 
                   session:session 
          completionHandler:^(ALTAppGroup *group, NSError *error) {
    if (error) {
        NSLog(@"Failed to add App Group: %@", error);
    } else {
        NSLog(@"Created App Group: %@", group.name);
        
        // Assign to App ID
        [api assignAppID:appID 
              toGroups:@[group] 
                  team:team 
               session:session 
      completionHandler:^(BOOL success, NSError *error) {
            if (error) {
                NSLog(@"Failed to assign App ID to group: %@", error);
            } else {
                NSLog(@"Assigned App ID to group");
            }
        }];
    }
}];
```

---

## Provisioning Profiles API

### Fetch Provisioning Profile

Downloads a provisioning profile for an App ID and device type.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/downloadTeamProvisioningProfile.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
    <key>DTDK_Platform</key>
    <string>ios</string>
</dict>
```

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>provisioningProfile</key>
    <dict>
        <key>provisioningProfileId</key>
        <string><profile ID></string>
        <key>status</key>
        <string>Active</string>
        <key>provisioningProfile</key>
        <data><base64 encoded profile></data>
    </dict>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Error Codes:**
- `8201`: App ID does not exist

### Delete Provisioning Profile

Deletes an existing provisioning profile.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/deleteProvisioningProfile.action?clientId=XABBG36SBA
```

**Request Parameters:**
```xml
<dict>
    <key>clientId</key>
    <string>XABBG36SBA</string>
    <key>protocolVersion</key>
    <string>QH65B2</string>
    <key>requestId</key>
    <string><UUID></string>
    <key>teamId</key>
    <string><team ID></string>
    <key>provisioningProfileId</key>
    <string><profile ID></string>
</dict>
```

**Error Codes:**
- `35`: Invalid provisioning profile identifier
- `8101`: Provisioning profile does not exist

### Usage Example (Objective-C)

```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api fetchProvisioningProfileForAppID:appID 
                           deviceType:ALTDeviceTypeiPhone 
                                 team:team 
                              session:session 
                 completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch provisioning profile: %@", error);
    } else {
        NSLog(@"Fetched provisioning profile: %@", profile.name);
        
        // Save profile
        [profile.data writeToFile:@"profile.mobileprovision" atomically:YES];
        
        // Delete if needed
        [api deleteProvisioningProfile:profile 
                               forTeam:team 
                               session:session 
                      completionHandler:^(BOOL success, NSError *error) {
            if (error) {
                NSLog(@"Failed to delete profile: %@", error);
            } else {
                NSLog(@"Deleted profile");
            }
        }];
    }
}];
```

---

## Entitlements and Capabilities

### Entitlements

Entitlements grant specific capabilities to applications:

- `ALTEntitlementApplicationIdentifier`: Bundle identifier
- `ALTEntitlementKeychainAccessGroups`: Keychain access groups
- `ALTEntitlementAppGroups`: App groups
- `ALTEntitlementGetTaskAllow`: Debugger attachment (development only)
- `ALTEntitlementIncreasedMemoryLimit`: Increased memory limit
- `ALTEntitlementTeamIdentifier`: Team identifier
- `ALTEntitlementInterAppAudio`: Inter-app audio
- `ALTEntitlementIncreasedDebuggingMemoryLimit`: Increased debugging memory limit
- `ALTEntitlementExtendedVirtualAddressing`: Extended virtual addressing

### Capabilities

Capabilities are features that can be enabled for App IDs:

- `ALTCapabilityIncreasedMemoryLimit`: Increased memory limit
- `ALTCapabilityIncreasedDebuggingMemoryLimit`: Increased debugging memory limit
- `ALTCapabilityExtendedVirtualAddressing`: Extended virtual addressing

### Features

Features map to entitlements:

- `ALTFeatureGameCenter`: Game Center
- `ALTFeatureAppGroups`: App Groups
- `ALTFeatureInterAppAudio`: Inter-app audio

### Free Developer Restrictions

Free developer accounts have restrictions on certain entitlements. Use `ALTFreeDeveloperCanUseEntitlement()` to check if an entitlement is available for free accounts.

---

## Complete Workflow Example

Here's a complete example of the entire workflow from authentication to provisioning profile generation:

```objective-c
// 1. Authenticate
ALTAnisetteData *anisetteData = /* obtain anisette data */;
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

[api authenticate:@"your@email.com"
          password:@"your-password"
      anisetteData:anisetteData
 verificationHandler:^(void (^completion)(NSString *)){
     // Handle 2FA
     NSString *code = /* get code from user */;
     completion(code);
 }
 completionHandler:^(ALTAccount *account, ALTAppleAPISession *session, NSError *error) {
     if (error) {
         NSLog(@"Authentication failed: %@", error);
         return;
     }
     
     // 2. Fetch teams
     [api fetchTeamsForAccount:account 
                        session:session 
               completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
         ALTTeam *team = teams.firstObject;
         
         // 3. Fetch devices
         [api fetchDevicesForTeam:team 
                            types:ALTDeviceTypeAll 
                          session:session 
                 completionHandler:^(NSArray<ALTDevice *> *devices, NSError *error) {
             
             // 4. Add certificate
             [api addCertificateWithMachineName:@"My Mac" 
                                        toTeam:team 
                                      session:session 
                           completionHandler:^(ALTCertificate *certificate, NSError *error) {
                 
                 // 5. Add App ID
                 [api addAppIDWithName:@"My App" 
                        bundleIdentifier:@"com.example.myapp" 
                                   team:team 
                                session:session 
                       completionHandler:^(ALTAppID *appID, NSError *error) {
                     
                     // 6. Fetch provisioning profile
                     [api fetchProvisioningProfileForAppID:appID 
                                                deviceType:ALTDeviceTypeiPhone 
                                                      team:team 
                                                   session:session 
                                      completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
                         if (error) {
                             NSLog(@"Failed: %@", error);
                         } else {
                             NSLog(@"Success! Profile: %@", profile.name);
                             // Use profile to sign app
                         }
                     }];
                 }];
             }];
         }];
     }];
 }];
```

---

## Notes and Best Practices

1. **Session Management**: Keep the `ALTAppleAPISession` object secure as it contains authentication tokens
2. **Error Handling**: Always check error codes and handle them appropriately
3. **Rate Limiting**: Apple may rate limit API requests; implement exponential backoff
4. **Certificate Storage**: Store certificates securely using encrypted P12 files
5. **Device Limits**: Free accounts are limited to 3 devices; paid accounts have higher limits
6. **App ID Limits**: Free accounts are limited to 10 App IDs
7. **Profile Expiration**: Provisioning profiles expire after 7 days for free accounts
8. **Anisette Data**: Must be obtained from a trusted device or service

---

## References

- [AltSign Repository](https://github.com/sidestore/AltSign)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [SRP Protocol RFC 5054](https://tools.ietf.org/html/rfc5054)
