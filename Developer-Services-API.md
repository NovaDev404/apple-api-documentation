# Developer Services API Documentation

## Overview

The Developer Services API is Apple's internal API for managing developer resources including teams, devices, certificates, App IDs, App Groups, and provisioning profiles. This API is used by Xcode and other developer tools.

## Base URLs

```
https://developerservices2.apple.com/services/QH65B2/  (Legacy API)
https://developerservices2.apple.com/services/v1/     (Modern REST API)
```

**Protocol Version:** QH65B2  
**Client ID:** XABBG36SBA

## Authentication

All requests to the Developer Services API require authentication via the GSA service. The authentication credentials are passed in HTTP headers:

### Required Headers

```http
Content-Type: text/x-xml-plist
User-Agent: Xcode
Accept: text/x-xml-plist
Accept-Language: en-us
X-Apple-App-Info: com.apple.gs.xcode.auth
X-Xcode-Version: 11.2 (11B41)
X-Apple-I-Identity-Id: <dsid from GSA authentication>
X-Apple-GS-Token: <auth token from GSA authentication>
X-Apple-I-MD-M: <machine ID from anisette data>
X-Apple-I-MD: <one-time password from anisette data>
X-Apple-I-MD-LU: <local user ID from anisette data>
X-Apple-I-MD-RINFO: <routing info from anisette data>
X-Mme-Device-Id: <device unique identifier from anisette data>
X-MMe-Client-Info: <device description from anisette data>
X-Apple-I-Client-Time: <current ISO 8601 date>
X-Apple-Locale: <locale from anisette data>
X-Apple-I-Locale: <locale from anisette data>
X-Apple-I-TimeZone: <timezone from anisette data>
```

## Request Format

### Legacy API (QH65B2)

Most endpoints use the legacy API with Property List (plist) format in the request body.

**Common Request Parameters:**
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
    <string><UUID in uppercase></string>
    <key>teamId</key>
    <string><team ID (optional)></string>
    <!-- Additional endpoint-specific parameters -->
</dict>
</plist>
```

### Modern REST API (v1)

Some endpoints use the modern REST API with JSON format.

**Common Headers for REST API:**
```http
Content-Type: application/json
Accept: application/json
```

## Response Format

### Legacy API Response

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key><endpoint-specific data></key>
    <dict>
        <!-- Response data -->
    </dict>
    <key>resultCode</key>
    <integer>0</integer>
    <key>userString</key>
    <string><error message if resultCode != 0></string>
</dict>
</plist>
```

**Result Codes:**
- `0`: Success
- Non-zero: Error (see endpoint-specific error codes)

### Modern REST API Response

```json
{
    "data": [
        {
            "id": "<resource ID>",
            "type": "<resource type>",
            "attributes": {
                <!-- Resource attributes -->
            }
        }
    ],
    "meta": {
        <!-- Metadata -->
    }
}
```

## Teams API

### Fetch Teams

Retrieves all teams associated with the authenticated account.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/listTeams.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
</dict>
</plist>
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
- `Individual`: Personal team (free developer)
- `Organization`: Paid organization team
- `Free`: Free developer team

**Error Codes:**
- No specific error codes for this endpoint

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api fetchTeamsForAccount:account 
                   session:session 
          completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch teams: %@", error);
        return;
    }
    
    for (ALTTeam *team in teams) {
        NSLog(@"Team: %@ (ID: %@, Type: %ld)", 
              team.name, team.identifier, (long)team.type);
    }
    
    // Select a team for subsequent operations
    ALTTeam *selectedTeam = teams.firstObject;
}];
```

## Devices API

### Fetch Devices

Retrieves all registered devices for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listDevices.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
</dict>
</plist>
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

**Device Classes:**
- `iPhone`: iPhone devices
- `iPad`: iPad devices
- `AppleTV`: Apple TV devices

**Error Codes:**
- No specific error codes for this endpoint

### Register Device

Registers a new device with the team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/addDevice.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>deviceNumber</key>
    <string><device UDID></string>
    <key>name</key>
    <string><device name></string>
    <key>DTDK_Platform</key>
    <string>ios</string>
    <key>subPlatform</key>
    <string><optional: tvOS for Apple TV></string>
</dict>
</plist>
```

**Platform Values:**
- `ios`: iPhone/iPad devices
- `tvos`: Apple TV devices (requires `subPlatform: tvOS`)

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>device</key>
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
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Error Codes:**
- `35`: Device already registered or invalid device ID

**Device Limits:**
- Free accounts: 3 devices
- Paid accounts: 100+ devices

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];
[api registerDeviceWithName:@"My iPhone" 
                 identifier:@"<device UDID>" 
                        type:ALTDeviceTypeiPhone 
                       team:team 
                    session:session 
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
```

## Certificates API

### Fetch Certificates (Modern REST API)

Retrieves all development certificates for a team using the modern REST API.

**Endpoint:**
```
GET https://developerservices2.apple.com/services/v1/certificates?filter[certificateType]=IOS_DEVELOPMENT
```

**Method:** GET

**Headers:**
```http
Content-Type: application/json
Accept: application/json
X-Apple-I-Identity-Id: <dsid>
X-Apple-GS-Token: <auth token>
<!-- Other common headers -->
```

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
                "certificateContent": "<base64 encoded DER certificate>",
                "expirationDate": "<ISO 8601 date>"
            }
        }
    ]
}
```

**Certificate Types:**
- `IOS_DEVELOPMENT`: iOS development certificate
- `IOS_DISTRIBUTION`: iOS distribution certificate
- `MAC_DEVELOPMENT`: Mac development certificate
- `MAC_DISTRIBUTION`: Mac distribution certificate

### Add Certificate

Creates a new development certificate by submitting a CSR (Certificate Signing Request).

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/submitDevelopmentCSR.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>csrContent</key>
    <string><base64 encoded CSR></string>
    <key>machineId</key>
    <string><machine UUID></string>
    <key>machineName</key>
    <string><machine name></string>
</dict>
</plist>
```

**CSR Generation:**
```objective-c
ALTCertificateRequest *request = [ALTCertificateRequest newRequest];
NSString *encodedCSR = [[NSString alloc] initWithData:request.data 
                                              encoding:NSUTF8StringEncoding];
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
        <data><base64 encoded DER certificate></data>
    </dict>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Error Codes:**
- `3250`: Invalid certificate request

**Certificate Limits:**
- Free accounts: Limited number of certificates
- Paid accounts: Higher limits

### Revoke Certificate (Modern REST API)

Revokes an existing certificate using the modern REST API.

**Endpoint:**
```
DELETE https://developerservices2.apple.com/services/v1/certificates/<certificate ID>
```

**Method:** DELETE

**Headers:** Same as fetch certificates

**Response:**
```json
{
    "data": {
        "id": "<certificate ID>",
        "type": "certificates"
    }
}
```

**Error Codes:**
- `7252`: Certificate does not exist

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

// Add certificate
[api addCertificateWithMachineName:@"My Mac" 
                              toTeam:team 
                            session:session 
                 completionHandler:^(ALTCertificate *certificate, NSError *error) {
    if (error) {
        NSLog(@"Failed to add certificate: %@", error);
        return;
    }
    
    NSLog(@"Created certificate: %@", certificate.name);
    
    // Save as encrypted P12
    NSData *p12Data = [certificate encryptedP12DataWithPassword:@"secure-password"];
    [p12Data writeToFile:@"certificate.p12" atomically:YES];
    
    // Revoke if needed
    [api revokeCertificate:certificate 
                   forTeam:team 
                   session:session 
          completionHandler:^(BOOL success, NSError *error) {
        if (error) {
            NSLog(@"Failed to revoke certificate: %@", error);
        } else {
            NSLog(@"Certificate revoked successfully");
        }
    }];
}];
```

## App IDs API

### Fetch App IDs

Retrieves all App IDs for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listAppIds.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
</dict>
</plist>
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
                <key>application-identifier</key>
                <string><bundle identifier></string>
                <key>keychain-access-groups</key>
                <array>
                    <string><bundle identifier>.*</string>
                </array>
                <key>com.apple.developer.team-identifier</key>
                <string><team identifier></string>
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

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>identifier</key>
    <string><bundle identifier></string>
    <key>name</key>
    <string><App ID name></string>
</dict>
</plist>
```

**Bundle Identifier Format:**
- Explicit: `com.example.myapp`
- Wildcard: `com.example.*`

**Name Sanitization:**
App ID names are sanitized to remove special characters and diacritics. Only alphanumeric characters and spaces are allowed.

**Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>appId</key>
    <dict>
        <key>appIdId</key>
        <string><App ID ID></string>
        <key>name</key>
        <string><sanitized App ID name></string>
        <key>identifier</key>
        <string><bundle identifier></string>
    </dict>
    <key>resultCode</key>
    <integer>0</integer>
</dict>
</plist>
```

**Error Codes:**
- `35`: Invalid App ID name
- `9120`: Maximum App ID limit reached
- `9401`: Bundle identifier unavailable
- `9412`: Invalid bundle identifier

**App ID Limits:**
- Free accounts: 10 App IDs
- Paid accounts: Unlimited

### Update App ID

Updates an existing App ID with features and entitlements.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/updateAppId.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
        <key>com.apple.developer.team-identifier</key>
        <string><team identifier></string>
        <!-- Additional entitlements -->
    </dict>
</dict>
</plist>
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

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
</dict>
</plist>
```

**Error Codes:**
- `9100`: App ID does not exist

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

// Add App ID
[api addAppIDWithName:@"My App" 
       bundleIdentifier:@"com.example.myapp" 
                  team:team 
               session:session 
      completionHandler:^(ALTAppID *appID, NSError *error) {
    if (error) {
        NSLog(@"Failed to add App ID: %@", error);
        return;
    }
    
    NSLog(@"Created App ID: %@", appID.name);
    
    // Update with entitlements
    NSMutableDictionary *entitlements = [appID.entitlements mutableCopy];
    entitlements[ALTEntitlementKeychainAccessGroups] = @[@"com.example.myapp.*"];
    appID.entitlements = entitlements;
    
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

## App Groups API

### Fetch App Groups

Retrieves all App Groups for a team.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/listApplicationGroups.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
</dict>
</plist>
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

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>identifier</key>
    <string><group identifier></string>
    <key>name</key>
    <string><group name></string>
</dict>
</plist>
```

**Group Identifier Format:**
Must start with `group.` followed by reverse-DNS style identifier:
- Valid: `group.com.example.myapp`
- Invalid: `com.example.myapp`

**Error Codes:**
- `35`: Invalid App Group

### Assign App ID to Groups

Assigns an App ID to one or more App Groups.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/assignApplicationGroupToAppId.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
    <key>applicationGroups</key>
    <array>
        <string><group ID></string>
    </array>
</dict>
</plist>
```

**Error Codes:**
- `9115`: App ID does not exist
- `35`: App Group does not exist

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

// Add App Group
[api addAppGroupWithName:@"My App Group" 
          groupIdentifier:@"group.com.example.myapp" 
                      team:team 
                   session:session 
          completionHandler:^(ALTAppGroup *group, NSError *error) {
    if (error) {
        NSLog(@"Failed to add App Group: %@", error);
        return;
    }
    
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
            NSLog(@"Assigned App ID to group successfully");
        }
    }];
}];
```

## Provisioning Profiles API

### Fetch Provisioning Profile

Downloads a provisioning profile for an App ID and device type.

**Endpoint:**
```
POST https://developerservices2.apple.com/services/QH65B2/ios/downloadTeamProvisioningProfile.action?clientId=XABBG36SBA
```

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>appIdId</key>
    <string><App ID ID></string>
    <key>DTDK_Platform</key>
    <string>ios</string>
    <key>subPlatform</key>
    <string><optional: tvOS for Apple TV></string>
</dict>
</plist>
```

**Platform Values:**
- `ios`: iPhone/iPad
- `tvos`: Apple TV (requires `subPlatform: tvOS`)

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
        <data><base64 encoded .mobileprovision file></data>
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

**Method:** POST

**Request Body:**
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
    <string><team ID></string>
    <key>provisioningProfileId</key>
    <string><profile ID></string>
</dict>
</plist>
```

**Error Codes:**
- `35`: Invalid provisioning profile identifier
- `8101`: Provisioning profile does not exist

**Profile Expiration:**
- Free accounts: 7 days
- Paid accounts: 1 year

**Usage Example (Objective-C):**
```objective-c
ALTAppleAPI *api = [ALTAppleAPI sharedAPI];

// Fetch provisioning profile
[api fetchProvisioningProfileForAppID:appID 
                           deviceType:ALTDeviceTypeiPhone 
                                 team:team 
                              session:session 
                 completionHandler:^(ALTProvisioningProfile *profile, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch provisioning profile: %@", error);
        return;
    }
    
    NSLog(@"Fetched provisioning profile: %@", profile.name);
    NSLog(@"Expiration: %@", profile.expirationDate);
    
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
            NSLog(@"Deleted profile successfully");
        }
    }];
}];
```

## Common Error Handling

### Result Code Handling

Most endpoints return a `resultCode` field:
- `0`: Success
- Non-zero: Error (check `userString` for message)

### Error Code Reference

| Code | Description | Common Cause |
|------|-------------|--------------|
| 0 | Success | - |
| 35 | General validation error | Invalid input, missing required fields |
| 9100 | App ID does not exist | Invalid App ID ID |
| 9115 | App ID does not exist | Invalid App ID ID in group assignment |
| 9120 | Maximum App ID limit reached | Too many App IDs for account type |
| 9401 | Bundle identifier unavailable | Bundle ID already in use |
| 9412 | Invalid bundle identifier | Malformed bundle ID |
| 8101 | Provisioning profile does not exist | Invalid profile ID |
| 8201 | App ID does not exist | Invalid App ID ID for profile |
| 3250 | Invalid certificate request | Malformed CSR |
| 7252 | Certificate does not exist | Invalid certificate ID |

### Error Handling Example

```objective-c
NSError *error = /* error from API call */;
if (error) {
    switch (error.code) {
        case ALTAppleAPIErrorNoTeams:
            NSLog(@"No teams available for account");
            break;
        case ALTAppleAPIErrorDeviceAlreadyRegistered:
            NSLog(@"Device is already registered");
            break;
        case ALTAppleAPIErrorInvalidDeviceID:
            NSLog(@"Invalid device UDID");
            break;
        case ALTAppleAPIErrorMaximumAppIDLimitReached:
            NSLog(@"Maximum App ID limit reached");
            break;
        case ALTAppleAPIErrorBundleIdentifierUnavailable:
            NSLog(@"Bundle identifier already in use");
            break;
        default:
            NSLog(@"Unknown error: %@", error);
            break;
    }
}
```

## Rate Limiting

Apple may rate limit API requests. Implement exponential backoff:

```objective-c
- (void)sendRequestWithRetry:(NSURLRequest *)request 
                 maxRetries:(NSInteger)maxRetries 
                  completion:(void(^)(NSData *data, NSError *error))completion {
    [self sendRequestWithRetry:request 
                    attempt:0 
                   maxRetries:maxRetries 
                   completion:completion];
}

- (void)sendRequestWithRetry:(NSURLRequest *)request 
                     attempt:(NSInteger)attempt 
                  maxRetries:(NSInteger)maxRetries 
                  completion:(void(^)(NSData *data, NSError *error))completion {
    NSURLSessionDataTask *task = [self.session dataTaskWithRequest:request 
                                                 completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (error && error.code == NSURLErrorTooManyRequests && attempt < maxRetries) {
            // Exponential backoff
            NSInteger delay = (NSInteger)pow(2, attempt);
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delay * NSEC_PER_SEC)), 
                          dispatch_get_main_queue(), ^{
                [self sendRequestWithRetry:request 
                                   attempt:attempt + 1 
                                maxRetries:maxRetries 
                                completion:completion];
            });
        } else {
            completion(data, error);
        }
    }];
    [task resume];
}
```

## Best Practices

1. **Session Management**: Keep `ALTAppleAPISession` secure; it contains authentication tokens
2. **Error Handling**: Always check error codes and handle appropriately
3. **Rate Limiting**: Implement exponential backoff for failed requests
4. **Certificate Storage**: Store certificates securely using encrypted P12 files
5. **Profile Expiration**: Monitor profile expiration dates and refresh before expiry
6. **Device Limits**: Be aware of device limits for your account type
7. **App ID Limits**: Be aware of App ID limits for your account type
8. **Request IDs**: Use unique UUIDs for each request

## References

- [AltSign Repository](https://github.com/sidestore/AltSign)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Property List Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/PropertyLists/)
