# SMS Verification Setup Guide

This document describes how SMS verification works in Souq-alhawameer and how to configure Firebase Phone Authentication.

## ⚠️ TROUBLESHOOTING: "Fail to Send" Error

**If you see "فشل إرسال رمز التحقق" (fail to send verification code), follow these steps:**

### Quick Fix Checklist

1. **✅ Enable Phone Authentication in Firebase Console** (Most Common Issue)
   - Go to https://console.firebase.google.com/
   - Select project: `souq-alhawameer`
   - Navigate to **Authentication** → **Sign-in method**
   - Find **Phone** provider and click on it
   - Toggle **Enable** and click **Save**
   - **This is the most common cause of "fail to send" errors!**

2. **✅ Check Browser Console for Error Details**
   - Press F12 to open Developer Tools
   - Go to Console tab
   - Look for errors starting with "🔍 Firebase Auth Error Details:"
   - Note the error code (e.g., `auth/operation-not-allowed`)

3. **✅ Verify Phone Number Format**
   - Must be Saudi phone number starting with `05`
   - Must be exactly 10 digits
   - Examples: `0501234567`, `0551234567`
   - Invalid: `501234567`, `+966501234567` (these are auto-converted)

4. **✅ Add Authorized Domain**
   - In Firebase Console: **Authentication** → **Settings** → **Authorized domains**
   - Add your domain (e.g., `localhost` for testing, your domain for production)

5. **✅ Enable Billing (For Production)**
   - Firebase Phone Auth requires billing enabled for production use
   - Free tier includes limited SMS sends per month
   - Go to Firebase Console → **Billing** to enable

### Common Error Codes and Solutions

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `auth/operation-not-allowed` | Phone Authentication not enabled | Enable Phone provider in Firebase Console |
| `auth/invalid-phone-number` | Wrong phone format | Use 05xxxxxxxx format (10 digits) |
| `auth/too-many-requests` | Rate limit exceeded | Wait 5 minutes and try again |
| `auth/quota-exceeded` | SMS quota reached | Enable billing or wait for quota reset |
| `auth/captcha-check-failed` | reCAPTCHA failed | Reload page and try again |
| No error code | Firebase not configured | Check Firebase setup steps below |

### Development Testing Without SMS

For development, you can add test phone numbers that don't send actual SMS:

1. Firebase Console → **Authentication** → **Sign-in method** → **Phone**
2. Scroll to **Phone numbers for testing**
3. Add: Phone `+966501234567`, Code `123456`
4. Use this number during registration - no SMS will be sent
5. Enter code `123456` to complete verification

## Overview

SMS verification is required for:
1. **New Student Registration** - Students must verify their phone number before account creation
2. **New Guest Registration** - Guests must verify their phone number before account creation
3. **Password Reset** - Users must verify their phone number to reset their password

## Firebase Phone Authentication Setup

### Prerequisites
- Firebase project already created and configured in `index.html`
- Firebase Authentication enabled

### Steps to Enable Phone Authentication

1. **Go to Firebase Console**
   - Navigate to https://console.firebase.google.com/
   - Select your project: `souq-alhawameer`

2. **Enable Phone Authentication**
   - Go to **Authentication** → **Sign-in method**
   - Click on **Phone** provider
   - Enable the **Phone** sign-in method
   - Click **Save**

3. **Configure reCAPTCHA (Important)**
   - The app uses invisible reCAPTCHA for bot protection
   - reCAPTCHA is automatically configured by Firebase
   - Ensure your domain is added to the authorized domains list:
     - Go to **Authentication** → **Settings** → **Authorized domains**
     - Add your domain (e.g., `souq-alhawameer.web.app` or your custom domain)

4. **SMS Provider Configuration**
   - Firebase automatically uses its built-in SMS provider
   - For production, you may need to enable billing in your Firebase project
   - Free tier includes limited SMS sends per month

5. **Testing Phone Authentication**
   - For development, you can add test phone numbers:
     - Go to **Authentication** → **Sign-in method** → **Phone**
     - Scroll down to **Phone numbers for testing**
     - Add test phone numbers (e.g., `+966501234567`) with test codes (e.g., `123456`)
   - Test numbers skip actual SMS sending but still validate the verification flow

## Implementation Details

### Verification Flow

#### 1. Registration Flow (Students & Guests)
```
User fills form → Send SMS via Firebase → User enters code → Verify code → Create account
```

- User submits registration form
- Firebase sends SMS with 6-digit code
- User redirected to verification screen
- User enters code
- Code verified via Firebase
- Account created with `phoneVerified: true` and `phoneVerifiedAt` timestamp
- User redirected to login

#### 2. Password Reset Flow
```
User enters phone → Send SMS via Firebase → User enters code → Verify code → Reset password
```

- User enters registered phone number
- Firebase sends SMS with 6-digit code
- User redirected to reset screen
- User enters code and new password
- Code verified via Firebase
- Password updated and `phoneVerified: true` flag set
- User redirected to login

### Security Features

1. **Phone Number Verification Tracking**
   - `phoneVerified` (boolean): Indicates if phone is verified
   - `phoneVerifiedAt` (ISO timestamp): When verification occurred

2. **Login Protection**
   - Users with `phoneVerified: false` cannot log in
   - Existing users without the flag can log in (backward compatible)

3. **Phone Number Matching**
   - Verification handlers check that Firebase-returned phone matches submitted phone
   - Prevents phone number substitution attacks

4. **Screen Navigation Protection**
   - Direct access to verification screens is blocked
   - Users must complete the proper flow

5. **SMS Retry Logic**
   - Automatic retry with exponential backoff
   - Up to 3 attempts for failed SMS sends
   - User-friendly error messages

### Verification Status

New accounts are created with:
```javascript
{
  phoneVerified: false,          // Initially false
  phoneVerifiedAt: undefined     // Initially undefined
}
```

After successful verification:
```javascript
{
  phoneVerified: true,                    // Set to true
  phoneVerifiedAt: "2026-02-17T21:00:00Z" // ISO timestamp
}
```

## Troubleshooting

### SMS Not Sending

1. **Check Firebase Console**
   - Verify Phone Authentication is enabled
   - Check Firebase project quota/billing

2. **Check Browser Console**
   - Look for Firebase error messages
   - Common errors:
     - `auth/quota-exceeded`: SMS quota exceeded
     - `auth/invalid-phone-number`: Phone format incorrect
     - `auth/too-many-requests`: Too many attempts

3. **Use Test Phone Numbers**
   - Add test numbers in Firebase Console for development
   - Test numbers bypass actual SMS sending

### Verification Code Not Working

1. **Code Expired**
   - Codes expire after a few minutes
   - User must request new code

2. **Wrong Code**
   - User must enter exact 6-digit code from SMS
   - Code is case-sensitive

3. **Phone Number Mismatch**
   - Verification checks Firebase phone matches submitted phone
   - Check console for mismatch errors

### Users Locked Out

If a user's account has `phoneVerified: false` and they can't log in:

1. **Option 1: Re-register**
   - User can register again (will require SMS verification)
   - Old unverified account will remain in database

2. **Option 2: Admin Manual Fix**
   - Admin can manually set `phoneVerified: true` in Firebase Console
   - Go to Firestore → users collection → find user → edit document

## Phone Number Format

The app expects Saudi phone numbers in format: `05xxxxxxxx`

- Must start with `05`
- Must be exactly 10 digits
- Internally converted to international format: `+9665xxxxxxxx`

Examples:
- Valid: `0501234567`, `0551234567`
- Invalid: `501234567`, `+966501234567`, `05012345678`

## Firebase Configuration

Current Firebase configuration in `index.html`:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyC_Tb2a3FklDJhd_zWg_sE5kAaz7XG8mHM",
  authDomain: "souq-alhawameer.firebaseapp.com",
  projectId: "souq-alhawameer",
  storageBucket: "souq-alhawameer.firebasestorage.app",
  messagingSenderId: "7274083119",
  appId: "1:7274083119:web:800b10fb25272d7348e7da"
};
```

## Testing Checklist

- [ ] Test guest registration with valid phone number
- [ ] Test student registration with valid phone number
- [ ] Test password reset with registered phone number
- [ ] Test login with verified account
- [ ] Test login with unverified account (should be blocked)
- [ ] Test invalid phone number format
- [ ] Test expired verification code
- [ ] Test wrong verification code
- [ ] Test direct navigation to verification screens (should be blocked)
- [ ] Test phone number already registered

## Security Considerations

1. **Client-Side Limitations**
   - This is a client-side only application
   - Window variables can be manipulated via console
   - However, actual SMS verification through Firebase cannot be bypassed

2. **Defense in Depth**
   - Multiple layers of security checks
   - Phone number matching validation
   - Screen navigation protection
   - Login verification status check

3. **Firebase Security**
   - Firebase handles actual SMS sending and verification
   - Firebase Auth tokens are secure
   - reCAPTCHA prevents bot attacks

## Support

For issues or questions:
- Check Firebase Console for authentication logs
- Check browser console for detailed error messages
- Review this documentation for troubleshooting steps
