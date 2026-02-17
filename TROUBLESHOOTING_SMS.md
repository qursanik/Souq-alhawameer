# 🚨 SMS Verification "Fail to Send" - Quick Fix

## Problem
Getting error: **"فشل إرسال رمز التحقق"** (fail to send verification code)

## Most Common Cause (90% of cases)
**Firebase Phone Authentication is NOT enabled!**

## Solution (5 minutes)

### Step 1: Open Firebase Console
Go to: https://console.firebase.google.com/

### Step 2: Select Your Project
Click on: **souq-alhawameer**

### Step 3: Enable Phone Authentication
1. Click **Authentication** in left menu
2. Click **Sign-in method** tab
3. Find **Phone** in the list
4. Click on **Phone**
5. Toggle **Enable** switch
6. Click **Save**

### Step 4: Test Again
- Go back to the app
- Reload the page (F5)
- Try registration again
- SMS should now send successfully!

---

## Still Not Working?

### Check Console Logs (Press F12)
Look for: `🔍 Firebase Auth Error Details:`

### Common Error Codes

**`auth/operation-not-allowed`**
- Phone Auth not enabled
- Follow steps above

**`auth/invalid-phone-number`**
- Wrong format
- Use: `05xxxxxxxx` (10 digits starting with 05)

**`auth/too-many-requests`**
- Too many attempts
- Wait 5 minutes

**`auth/quota-exceeded`**
- SMS limit reached
- Enable billing in Firebase Console

**`auth/captcha-check-failed`**
- reCAPTCHA failed
- Reload page (F5) and try again

---

## Development Testing (No SMS)

Add test phone numbers in Firebase Console:

1. **Authentication** → **Sign-in method** → **Phone**
2. Scroll to **Phone numbers for testing**
3. Add:
   - Phone: `+966501234567`
   - Code: `123456`
4. Use this number during registration
5. Enter code `123456` - no SMS sent!

---

## Need More Help?

See complete guide: `SMS_VERIFICATION_SETUP.md`

Or check console logs for specific error details.
