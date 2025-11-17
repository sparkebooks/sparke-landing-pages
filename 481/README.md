# Landing Page Tracking Documentation

## Overview

This landing page implements comprehensive user tracking and data collection before redirecting users to the target application. It captures device information, user behavior, and campaign parameters to enable attribution and user analytics.

## Book ID Configuration

The book ID is extracted from the landing page filename. For example:
- Filename: `3106.html` → Book ID: `3106`
- This book ID should be passed as a URL parameter when users arrive at the landing page
- Example: `https://yourdomain.com/landers/481/3106.html?book_id=3106&fbclid=...`

## Data Collection Methods

### 1. Clipboard Data Transfer

When the CTA button is clicked, the landing page copies an encoded Base64 string to the user's clipboard containing all tracking data. This allows the app to read comprehensive tracking information even if URL parameters are truncated.

**Clipboard Data Structure (Base64 encoded JSON):**
```json
{
  // Basic tracking data
  "timestamp": 1234567890,
  "sessionId": "session_1234567890_abc123",
  "pageUrl": "full URL of the landing page",
  "referrer": "referring URL or 'direct'",
  
  // Device & browser information
  "userAgent": "full user agent string",
  "language": "en-US",
  "timezone": "America/New_York",
  "screenResolution": "1920x1080",
  "viewportSize": "1920x937",
  "deviceMemory": 8,
  "hardwareConcurrency": 4,
  "platform": "MacIntel",
  "cookieEnabled": true,
  "doNotTrack": "1",
  "deviceType": "mobile/desktop",
  "os": "android/ios/unknown",
  
  // IP and location data (from ipapi.co)
  "ip": "user IP address",
  "country": "United States",
  "countryCode": "US",
  "region": "California",
  "city": "San Francisco",
  "postal": "94105",
  "latitude": 37.7749,
  "longitude": -122.4194,
  "isp": "Internet Service Provider",
  
  // Facebook tracking parameters
  "fbclid": "Facebook click ID",
  "ad_id": "Facebook ad ID",
  "adset_id": "Facebook ad set ID",
  "campaign_id": "Facebook campaign ID",
  "creative_id": "Facebook creative ID",
  "placement": "Facebook placement",
  "site_source_name": "fb",
  
  // UTM parameters
  "utm_source": "facebook",
  "utm_medium": "cpc",
  "utm_campaign": "campaign name",
  "utm_content": "ad content identifier",
  "utm_term": "targeting terms",
  
  // Facebook cookies
  "fbc": "_fbc cookie value",
  "fbp": "_fbp cookie value",
  
  // Device fingerprint (FingerprintJS)
  "fingerprint": "unique visitor ID",
  "fingerprintConfidence": 0.995,
  "fingerprintComponents": {
    "fonts": ["Arial", "Helvetica", ...],
    "audio": 35.73833402246237,
    "canvas": "canvas fingerprint hash",
    "webgl": "webgl fingerprint data",
    "timezone": "America/New_York",
    "languages": ["en-US", "en"],
    "colorDepth": 24,
    "deviceMemory": 8,
    "hardwareConcurrency": 4,
    "screenResolution": [1920, 1080]
  },
  
  // All URL parameters
  "allUrlParams": {
    "book_id": "3106",
    "fbclid": "...",
    "utm_source": "...",
    // ... any other parameters
  }
}
```

### 2. URL Parameters

The landing page appends tracking parameters to the CTA button URL for deferred deep linking:

**Parameters added to CTA URL:**
- `book_id`: The book ID (extracted from landing page URL or filename)
- `fbclid`: Facebook click ID
- `ad_id`: Facebook ad ID
- `adset_id`: Facebook ad set ID
- `campaign_id`: Facebook campaign ID
- `utm_source`: UTM source
- `utm_medium`: UTM medium
- `utm_campaign`: UTM campaign
- `fingerprint`: Unique visitor ID from FingerprintJS
- `fp_confidence`: Fingerprint confidence score
- `tracking_data`: Base64 encoded full tracking payload
- `session_id`: Unique session identifier
- `landing_page`: Landing page identifier (currently hardcoded as '481')

**Example final URL:**
```
https://9bik5.app.link/lp1?book_id=3106&fbclid=IwAR...&ad_id=123&adset_id=456&campaign_id=789&utm_source=facebook&utm_medium=cpc&utm_campaign=summer_sale&fingerprint=abc123def456&fp_confidence=0.995&tracking_data=eyJ0aW1lc3RhbXAiOj...&session_id=session_1234567890_abc123&landing_page=481
```

## Implementation Details

### Scripts and Libraries

1. **FingerprintJS v4**: Generates unique device fingerprints for user identification
2. **Meta Pixel (Facebook Pixel)**: Tracks page views and CTA clicks
3. **IP Geolocation**: Uses ipapi.co to fetch user location data

### Key Functions

1. **`getUrlParams()`**: Extracts all URL parameters
2. **`getFacebookParams()`**: Extracts Facebook-specific tracking parameters
3. **`getIPData()`**: Fetches IP and location data from ipapi.co
4. **`generateFingerprint()`**: Creates device fingerprint using FingerprintJS
5. **`createTrackingPayload()`**: Combines all tracking data
6. **`handleCTAClick()`**: Processes CTA click, copies data to clipboard, and appends URL parameters

### Data Flow

1. **Page Load:**
   - Extract all URL parameters
   - Initialize device fingerprinting
   - Fetch IP/location data
   - Combine all data into tracking payload
   - Store in localStorage and global variables

2. **CTA Click:**
   - Fire Facebook Pixel event
   - Copy encoded tracking data to clipboard
   - Append all relevant parameters to the CTA URL
   - Allow navigation to proceed

### Security Considerations

- All data is encoded in Base64 (not encrypted)
- Sensitive data should not be passed through URL parameters
- IP geolocation may fail due to ad blockers or network restrictions
- Clipboard access requires HTTPS in modern browsers

### Testing Parameters

To test the landing page with all parameters:
```
https://yourdomain.com/landers/481/3106.html?book_id=3106&fbclid=IwAR123&ad_id=123456&adset_id=789012&campaign_id=345678&creative_id=901234&placement=mobile_feed&utm_source=facebook&utm_medium=cpc&utm_campaign=test_campaign&utm_content=variant_a&utm_term=romance_readers
```

### App Integration

The receiving app should:
1. First attempt to read clipboard data for comprehensive tracking information
2. Fall back to URL parameters if clipboard is unavailable
3. Parse the Base64 encoded `tracking_data` parameter for full details
4. Use the `book_id` parameter to identify which book to display
5. Store tracking data for attribution and analytics

### Important Notes

- The landing page identifier is currently hardcoded as '481' in the `handleCTAClick` function (line 322)
- The book_id must be passed as a URL parameter to the landing page
- Facebook cookies (_fbc, _fbp) are only available if the user has previously interacted with Facebook
- Device fingerprinting may be blocked by privacy-focused browsers
- The clipboard API requires user interaction (click) to work properly