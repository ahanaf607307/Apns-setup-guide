# 🍎 Apple Push Notification Service (APNs) - Complete Integration Guide

This guide is for developers who need to set up automatic "real-time" updates for Apple Wallet passes.

---

## ❓ 1. Why is this needed? (The Purpose)
By default, an Apple Wallet pass is "static." If a user earns points on your backend, the card on their iPhone will **not** change. 

**APNs is the "Remote Control"** that allows your backend to tell the iPhone: *"Hey! Something changed for this user. Download the latest version of this card right now."*

Without APNs, your users will see outdated points, and your loyalty system will feel broken.

---

## 🔑 2. What data/keys are needed? (The Components)

You need **three specific pieces of data** from Apple to make this work:

1.  **Auth Key File (`.p8`)**: This is the digital key that allows your server to talk to Apple's Push servers.
2.  **Key ID**: A 10-character code unique to that key file.
3.  **Team ID**: A 10-character code for your entire Apple Developer account.
4.  **Pass Type ID**: The identifier for your specific card (e.g., `pass.com.belbeda.loyalty`).

---

## � 3. Where to find these values? (Step-by-Step)

### A. Get the .p8 Key and Key ID
1. Log in to [developer.apple.com](https://developer.apple.com/).
2. Click on **Certificates, Identifiers & Profiles**.
3. Click on **Keys** in the left sidebar.
4. Click the blue **plus (+)** icon.
5. Name it: `Apple Wallet Push Key`.
6. **Enable**: Check the box for **Apple Push Notifications service (APNs)**.
7. Click **Continue** -> **Register**.
8. **Download**: You will get a file like `AuthKey_ABC123.p8`. 
   > **CAUTION**: You can only download this file ONCE. Apple will not let you download it again.
9. **Key ID**: On that same screen, you will see a string like `ABC123DEFG`. **Copy this.**

### B. Get your Team ID
1. In your Apple Developer portal, click on the **Account** tab.
2. Scroll to **Membership Details**.
3. Your **Team ID** (e.g., `XYZ7890123`) is listed there.

---

## 🛠️ 4. How to add it to your project? (Implementation)

### Step 1: Upload the file
Upload your `AuthKey_XXXX.p8` file to your server's root folder or a `certs/` folder.

### Step 2: Configure .env
Add these values to your `.env` file so the backend can find them:

```bash
# APNs Key ID from Apple Portal
APPLE_APNS_KEY_ID=ABC123DEFG

# Path to the file you uploaded in Step 1
APPLE_APNS_KEY_PATH=./AuthKey_ABC123DEFG.p8

# Your Apple Team ID
APPLE_TEAM_ID=XYZ7890123

# Your Pass Type ID (created in Identifiers)
APPLE_PASS_TYPE_ID=pass.com.belbeda.loyalty
```

### Step 3: Deployment Checklist
-   **HTTPS**: Ensure your server has an SSL certificate. Apple will refuse to send pushes to an `http` server.
-   **Topic**: In the code (`apns.service.js`), we set the `notification.topic` to your `APPLE_PASS_TYPE_ID`. This is correct for Wallet.

---

## 🔄 5. The Process Flow (For Troubleshooting)
If updates aren't working, here is what is happening under the hood:
1.  **Staff adds points** -> Backend calls `apnsService.sendPassUpdatePush(pushToken)`.
2.  **Backend** uses the `.p8` key and `Key ID` to sign a request to Apple.
3.  **Apple (APNs)** sends a silent ping to the user's iPhone.
4.  **iPhone** wakes up and calls your URL: `/api/customer/apple-wallet-wws/v1/devices/...` to get the new points.
