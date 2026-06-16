# Troubleshooting Guide

This guide helps you diagnose and resolve common issues when working with the Apple APIs documented in this repository.

## Authentication Issues

### Problem: Authentication Failed with Error -20101 or -22406

**Error Message:** "Incorrect credentials"

**Cause:** Invalid Apple ID or password

**Solutions:**
1. Verify your Apple ID and password are correct
2. Ensure the Apple ID is lowercase (API requires lowercase)
3. Check if your account is locked or requires password reset
4. Verify you're using the correct Apple ID (not an email alias)

**Code Example:**
```objective-c
// Always lowercase the Apple ID
NSString *sanitizedAppleID = [appleID lowercaseString];

[api authenticate:sanitizedAppleID
          password:password
      anisetteData:anisetteData
 verificationHandler:nil
 completionHandler:^(ALTAccount *account, ALTAppleAPISession *session, NSError *error) {
    if (error.code == ALTAppleAPIErrorIncorrectCredentials) {
        NSLog(@"Please check your Apple ID and password");
    }
}];
```

### Problem: Authentication Failed with Error -22421

**Error Message:** "Invalid anisette data"

**Cause:** Anisette data is invalid, expired, or malformed

**Solutions:**
1. Ensure all required anisette fields are present
2. Verify the anisette data is current (not expired)
3. Check that device description matches a real Apple device
4. Try obtaining fresh anisette data from a different source

**Required Anisette Fields:**
```objective-c
// Verify all fields are set
NSAssert(anisetteData.machineID != nil, @"machineID is required");
NSAssert(anisetteData.oneTimePassword != nil, @"oneTimePassword is required");
NSAssert(anisetteData.localUserID != nil, @"localUserID is required");
NSAssert(anisetteData.deviceUniqueIdentifier != nil, @"deviceUniqueIdentifier is required");
NSAssert(anisetteData.deviceSerialNumber != nil, @"deviceSerialNumber is required");
NSAssert(anisetteData.deviceDescription != nil, @"deviceDescription is required");
NSAssert(anisetteData.date != nil, @"date is required");
NSAssert(anisetteData.locale != nil, @"locale is required");
NSAssert(anisetteData.timeZone != nil, @"timeZone is required");
```

### Problem: 2FA Required but No Handler Provided

**Error Message:** "Requires two-factor authentication"

**Cause:** Account has 2FA enabled but no verification handler was provided

**Solutions:**
1. Always provide a verification handler in the authenticate call
2. Handle both SMS and trusted device 2FA methods
3. Prompt the user for the 2FA code appropriately

**Code Example:**
```objective-c
[api authenticate:appleID
          password:password
      anisetteData:anisetteData
 verificationHandler:^(void (^completion)(NSString *)) {
     // Always provide a handler
     dispatch_async(dispatch_get_main_queue(), ^{
         NSString *code = [self promptUserFor2FACode];
         completion(code);
     });
 }
 completionHandler:^(ALTAccount *account, ALTAppleAPISession *session, NSError *error) {
    // Handle result
}];
```

### Problem: Incorrect Verification Code (2FA)

**Error Message:** "Incorrect verification code"

**Cause:** Invalid 2FA code entered

**Solutions:**
1. Verify the code was entered correctly
2. Ensure the code hasn't expired (codes expire quickly)
3. Request a new code if needed
4. Check if you're using the correct 2FA method (SMS vs trusted device)

## Team Issues

### Problem: No Teams Available

**Error Message:** "No teams available for account"

**Cause:** Account has no development teams

**Solutions:**
1. Ensure you've accepted the Apple Developer Program agreement
2. Check if your account is in good standing
3. Verify you're using the correct Apple ID
4. For free accounts, ensure you've enrolled in the free developer program

**Code Example:**
```objective-c
[api fetchTeamsForAccount:account 
                   session:session 
          completionHandler:^(NSArray<ALTTeam *> *teams, NSError *error) {
    if (error.code == ALTAppleAPIErrorNoTeams) {
        NSLog(@"No teams available. Please enroll in the developer program.");
        return;
    }
    
    if (teams.count == 0) {
        NSLog(@"No teams found. Check your developer account status.");
        return;
    }
    
    // Proceed with first team
    ALTTeam *team = teams.firstObject;
}];
```

## Device Issues

### Problem: Device Already Registered

**Error Message:** "Device already registered"

**Cause:** Device UDID is already registered to the team

**Solutions:**
1. Check if the device is already in your developer account
2. Use the existing device instead of trying to register again
3. If you need to replace it, remove the old device first (via Apple Developer portal)

**Code Example:**
```objective-c
[api registerDeviceWithName:name 
                 identifier:udid 
                        type:ALTDeviceTypeiPhone 
                       team:team 
                    session:session 
          completionHandler:^(ALTDevice *device, NSError *error) {
    if (error.code == ALTAppleAPIErrorDeviceAlreadyRegistered) {
        NSLog(@"Device already registered. Fetching existing device...");
        [self fetchExistingDevice:udid];
    } else if (error) {
        NSLog(@"Failed to register device: %@", error);
    }
}];
```

### Problem: Invalid Device ID

**Error Message:** "Invalid device ID"

**Cause:** Device UDID is malformed or invalid

**Solutions:**
1. Verify the UDID is a valid 40-character hexadecimal string
2. Ensure you're using the correct UDID (not serial number or other identifier)
3. Check for typos or extra characters

**Valid UDID Format:**
```
XXXXXXXX-XXXXXXXX-XXXXXXXX-XXXXXXXX-XXXXXXXX (40 hex characters)
```

### Problem: Device Limit Reached

**Error Message:** "Device limit reached"

**Cause:** Too many devices registered for the account type

**Solutions:**
1. Free accounts: Limit of 3 devices
2. Paid accounts: Limit of 100+ devices
3. Remove unused devices via Apple Developer portal
4. Upgrade to a paid account if needed

## Certificate Issues

### Problem: Invalid Certificate Request

**Error Message:** "Invalid certificate request (3250)"

**Cause:** CSR (Certificate Signing Request) is malformed or invalid

**Solutions:**
1. Ensure the CSR is properly generated
2. Verify the CSR is base64 encoded correctly
3. Check that the machine ID and name are valid
4. Try generating a new CSR

**Code Example:**
```objective-c
ALTCertificateRequest *request = [ALTCertificateRequest newRequest];
if (!request) {
    NSLog(@"Failed to generate certificate request");
    return;
}

NSString *encodedCSR = [[NSString alloc] initWithData:request.data 
                                              encoding:NSUTF8StringEncoding];
NSLog(@"CSR length: %lu", (unsigned long)encodedCSR.length);
```

### Problem: Certificate Does Not Exist

**Error Message:** "Certificate does not exist (7252)"

**Cause:** Attempting to revoke or access a non-existent certificate

**Solutions:**
1. Verify the certificate ID is correct
2. Fetch the list of certificates to find the correct ID
3. Check if the certificate was already revoked

**Code Example:**
```objective-c
// Fetch certificates first to get valid IDs
[api fetchCertificatesForTeam:team 
                       session:session 
          completionHandler:^(NSArray<ALTCertificate *> *certificates, NSError *error) {
    if (error) {
        NSLog(@"Failed to fetch certificates: %@", error);
        return;
    }
    
    // Find the certificate you want to revoke
    ALTCertificate *targetCertificate = nil;
    for (ALTCertificate *cert in certificates) {
        if ([cert.serialNumber isEqualToString:targetSerialNumber]) {
            targetCertificate = cert;
            break;
        }
    }
    
    if (!targetCertificate) {
        NSLog(@"Certificate not found");
        return;
    }
    
    // Now revoke it
    [api revokeCertificate:targetCertificate 
                   forTeam:team 
                   session:session 
          completionHandler:^(BOOL success, NSError *error) {
        // Handle result
    }];
}];
```

## App ID Issues

### Problem: Invalid App ID Name

**Error Message:** "Invalid App ID name (35)"

**Cause:** App ID name contains invalid characters or is too long

**Solutions:**
1. Use only alphanumeric characters and spaces
2. Remove special characters and diacritics
3. Keep the name under 50 characters
4. The API automatically sanitizes names, but may reject some

**Name Sanitization:**
```objective-c
// AltSign automatically sanitizes names
NSMutableCharacterSet *allowedCharacters = [[NSCharacterSet asciiAlphanumericCharacterSet] mutableCopy];
[allowedCharacters formUnionWithCharacterSet:[NSCharacterSet whitespaceCharacterSet]];

NSString *sanitizedName = [name stringByFoldingWithOptions:NSDiacriticInsensitiveSearch locale:nil];
sanitizedName = [[sanitizedName componentsSeparatedByCharactersInSet:[allowedCharacters invertedSet]] componentsJoinedByString:@""];
```

### Problem: Maximum App ID Limit Reached

**Error Message:** "Maximum App ID limit reached (9120)"

**Cause:** Too many App IDs for the account type

**Solutions:**
1. Free accounts: Limit of 10 App IDs
2. Paid accounts: Unlimited App IDs
3. Delete unused App IDs
4. Use wildcard bundle identifiers when possible
5. Upgrade to a paid account if needed

**Code Example:**
```objective-c
[api addAppIDWithName:name 
       bundleIdentifier:bundleIdentifier 
                  team:team 
               session:session 
      completionHandler:^(ALTAppID *appID, NSError *error) {
    if (error.code == ALTAppleAPIErrorMaximumAppIDLimitReached) {
        NSLog(@"Maximum App ID limit reached. Please delete unused App IDs.");
        [self listAndDeleteUnusedAppIDs];
    } else if (error) {
        NSLog(@"Failed to add App ID: %@", error);
    }
}];
```

### Problem: Bundle Identifier Unavailable

**Error Message:** "Bundle identifier unavailable (9401)"

**Cause:** Bundle identifier is already in use by another app

**Solutions:**
1. Choose a different bundle identifier
2. Check if you already have an App ID with this identifier
3. Use your own reverse domain name (e.g., com.yourcompany.appname)
4. Don't use bundle identifiers from other developers

### Problem: Invalid Bundle Identifier

**Error Message:** "Invalid bundle identifier (9412)"

**Cause:** Bundle identifier is malformed

**Solutions:**
1. Use reverse-DNS format: com.company.appname
2. Use only alphanumeric characters and periods
3. Don't start or end with a period
4. Don't use consecutive periods
5. Keep each segment between 1-20 characters

**Valid Bundle Identifiers:**
```
com.example.myapp
com.example.myapp.extension
com.example.*
```

**Invalid Bundle Identifiers:**
```
.myapp
com..example
com.example.myapp.
com.example.my_app
```

## App Group Issues

### Problem: Invalid App Group

**Error Message:** "Invalid App Group (35)"

**Cause:** App group identifier is malformed

**Solutions:**
1. App group identifiers must start with "group."
2. Use reverse-DNS format after "group."
3. Use only alphanumeric characters and periods
4. Don't use special characters or spaces

**Valid App Group Identifiers:**
```
group.com.example.myapp
group.com.example.*
```

**Invalid App Group Identifiers:**
```
com.example.myapp
group.com.example.my_app
group.com.example.my app
```

### Problem: App Group Does Not Exist

**Error Message:** "App Group does not exist (35)"

**Cause:** Attempting to assign a non-existent App Group to an App ID

**Solutions:**
1. Verify the App Group ID is correct
2. Fetch the list of App Groups to find the correct ID
3. Create the App Group first before assigning

## Provisioning Profile Issues

### Problem: Provisioning Profile Does Not Exist

**Error Message:** "Provisioning profile does not exist (8101)"

**Cause:** Attempting to delete a non-existent profile

**Solutions:**
1. Verify the profile ID is correct
2. Check if the profile was already deleted
3. Fetch a new profile instead of trying to access an old one

### Problem: App ID Does Not Exist for Profile

**Error Message:** "App ID does not exist (8201)"

**Cause:** App ID used for profile doesn't exist

**Solutions:**
1. Verify the App ID ID is correct
2. Ensure the App ID hasn't been deleted
3. Create the App ID first before fetching a profile

### Problem: Profile Expired

**Symptom:** App fails to install or run with profile expiration error

**Cause:** Provisioning profile has expired

**Solutions:**
1. Free accounts: Profiles expire after 7 days
2. Paid accounts: Profiles expire after 1 year
3. Fetch a new provisioning profile before the old one expires
4. Monitor expiration dates and refresh proactively

**Code Example:**
```objective-c
// Check profile expiration
ALTProvisioningProfile *profile = [[ALTProvisioningProfile alloc] initWithData:profileData];
NSDate *expirationDate = profile.expirationDate;
NSDate *now = [NSDate date];

NSTimeInterval timeUntilExpiration = [expirationDate timeIntervalSinceDate:now];
if (timeUntilExpiration < 86400 * 2) { // Less than 2 days
    NSLog(@"Profile expiring soon. Refreshing...");
    [self refreshProvisioningProfile];
}
```

## Network Issues

### Problem: Connection Timeout

**Error Message:** "The request timed out"

**Cause:** Network connectivity issues or server unresponsiveness

**Solutions:**
1. Check your internet connection
2. Verify Apple's services are operational (check Apple System Status)
3. Implement retry logic with exponential backoff
4. Increase timeout values if needed

**Retry Logic Example:**
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
        if (error && (error.code == NSURLErrorTimedOut || error.code == NSURLErrorNotConnectedToInternet) && attempt < maxRetries) {
            NSInteger delay = (NSInteger)pow(2, attempt);
            NSLog(@"Request failed, retrying in %ld seconds...", (long)delay);
            
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

### Problem: Rate Limiting

**Error Message:** "Too many requests" or HTTP 429

**Cause:** Sending too many requests in a short time

**Solutions:**
1. Implement exponential backoff
2. Add delays between requests
3. Cache responses when possible
4. Use batch operations instead of individual requests

## General Debugging Tips

### Enable Logging

```objective-c
// Enable detailed logging
#ifdef DEBUG
    NSLog(@"Request URL: %@", request.URL);
    NSLog(@"Request Headers: %@", request.allHTTPHeaderFields);
    NSLog(@"Request Body: %@", [[NSString alloc] initWithData:request.HTTPBody encoding:NSUTF8StringEncoding]);
#endif
```

### Validate Input

```objective-c
// Always validate input before sending requests
- (BOOL)validateBundleIdentifier:(NSString *)bundleIdentifier {
    if (!bundleIdentifier || bundleIdentifier.length == 0) {
        return NO;
    }
    
    // Check format
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"^[a-zA-Z0-9]+(\\.[a-zA-Z0-9]+)+$" 
                                                                           options:0 
                                                                             error:nil];
    NSTextCheckingResult *match = [regex firstMatchInString:bundleIdentifier 
                                                   options:0 
                                                     range:NSMakeRange(0, bundleIdentifier.length)];
    
    return match != nil;
}
```

### Check Response Codes

```objective-c
// Always check response codes
NSURLSessionDataTask *task = [session dataTaskWithRequest:request 
                                       completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
    NSLog(@"HTTP Status Code: %ld", (long)httpResponse.statusCode);
    
    if (httpResponse.statusCode >= 400) {
        NSLog(@"HTTP Error: %ld", (long)httpResponse.statusCode);
        // Handle error
    }
}];
```

### Save Failed Requests for Debugging

```objective-c
// Save failed request/response for debugging
- (void)saveFailedRequest:(NSURLRequest *)request response:(NSData *)response error:(NSError *)error {
    NSString *timestamp = [NSString stringWithFormat:@"%.0f", [[NSDate date] timeIntervalSince1970]];
    NSString *filename = [NSString stringWithFormat:@"failed_request_%@.txt", timestamp];
    
    NSMutableString *debugInfo = [NSMutableString string];
    [debugInfo appendFormat:@"URL: %@\n", request.URL];
    [debugInfo appendFormat:@"Method: %@\n", request.HTTPMethod];
    [debugInfo appendFormat:@"Headers: %@\n", request.allHTTPHeaderFields];
    [debugInfo appendFormat:@"Body: %@\n", [[NSString alloc] initWithData:request.HTTPBody encoding:NSUTF8StringEncoding]];
    [debugInfo appendFormat:@"Error: %@\n", error];
    [debugInfo appendFormat:@"Response: %@\n", [[NSString alloc] initWithData:response encoding:NSUTF8StringEncoding]];
    
    [debugInfo writeToFile:filename atomically:YES encoding:NSUTF8StringEncoding error:nil];
}
```

## Getting Help

If you're still experiencing issues after trying these solutions:

1. **Check the AltSign Repository**: [https://github.com/sidestore/AltSign](https://github.com/sidestore/AltSign)
2. **Search Existing Issues**: Check if someone else has encountered the same problem
3. **Create a Minimal Reproducible Example**: Simplify your code to the minimum needed to reproduce the issue
4. **Include Debug Information**: Provide logs, error messages, and steps to reproduce
5. **Check Apple System Status**: Verify Apple's services are operational

## Common Error Code Reference

| Error Code | Description | API |
|------------|-------------|-----|
| -20101 | Incorrect credentials | GSA |
| -22406 | Incorrect credentials | GSA |
| -22421 | Invalid anisette data | GSA |
| -21669 | Incorrect verification code | GSA |
| 35 | General validation error | Developer Services |
| 9100 | App ID does not exist | App IDs |
| 9115 | App ID does not exist | App Groups |
| 9120 | Maximum App ID limit reached | App IDs |
| 9401 | Bundle identifier unavailable | App IDs |
| 9412 | Invalid bundle identifier | App IDs |
| 8101 | Provisioning profile does not exist | Provisioning Profiles |
| 8201 | App ID does not exist | Provisioning Profiles |
| 3250 | Invalid certificate request | Certificates |
| 7252 | Certificate does not exist | Certificates |

## References

- [AltSign Repository](https://github.com/sidestore/AltSign)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Apple System Status](https://developer.apple.com/system-status/)
