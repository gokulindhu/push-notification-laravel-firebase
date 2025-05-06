# 🔔 Push Notification with Firebase in Laravel

This guide explains how to integrate **Firebase Cloud Messaging (FCM)** push notifications into a Laravel application.

---

## 🚀 Prerequisites

- Laravel 8 or later
- Firebase Project ([console](https://console.firebase.google.com/))
- Device tokens from mobile or web apps using Firebase SDK

---

## 🔧 Step 1: Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create a new project or use an existing one.
3. Go to **Project Settings > Cloud Messaging**
4. Copy your:
   - **Server Key (Legacy)**
   - **Sender ID**

---

## ⚙️ Step 2: Laravel Configuration

### Install Dependencies

```bash
composer require guzzlehttp/guzzle
# or for advanced Firebase features
composer require kreait/firebase-php
```

### Add Firebase Key to `.env`

```
FCM_SERVER_KEY=your_firebase_server_key_here
```

---

## 🛠 Step 3: Notification Service

Create a service to send push notifications.

### `app/Services/FirebaseNotificationService.php`

```php
namespace App\Services;

use Illuminate\Support\Facades\Http;

class FirebaseNotificationService
{
    protected $fcmUrl = 'https://fcm.googleapis.com/fcm/send';

    public function sendNotification($deviceToken, $title, $body, $data = [])
    {
        $response = Http::withHeaders([
            'Authorization' => 'key=' . env('FCM_SERVER_KEY'),
            'Content-Type' => 'application/json',
        ])->post($this->fcmUrl, [
            'to' => $deviceToken,
            'notification' => [
                'title' => $title,
                'body' => $body,
            ],
            'data' => $data,
            'priority' => 'high',
        ]);

        return $response->json();
    }
}
```

---

## 📡 Step 4: Use the Service in a Controller

```php
use App\Services\FirebaseNotificationService;

public function sendPush(FirebaseNotificationService $firebase)
{
    $deviceToken = 'user_device_token_here';
    $firebase->sendNotification($deviceToken, 'Hello!', 'This is a test notification.');
}
```

---

## 💾 Optional: Store Device Tokens

### Create Migration

```bash
php artisan make:model DeviceToken -m
```

#### Migration Example

```php
Schema::create('device_tokens', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('device_token');
    $table->timestamps();
});
```

### Example Model

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class DeviceToken extends Model
{
    protected $fillable = ['user_id', 'device_token'];
}
```

---

## 📱 Client-Side Setup

Use the Firebase SDK on your mobile or web app to:

1. Request permission for notifications
2. Retrieve the device token
3. Send that token to your Laravel backend

---

## ✅ Example Firebase SDK (JavaScript - Web)

```javascript
import { getMessaging, getToken, onMessage } from "firebase/messaging";
import { initializeApp } from "firebase/app";

const firebaseConfig = {
  apiKey: "your_api_key",
  authDomain: "your_project.firebaseapp.com",
  projectId: "your_project_id",
  messagingSenderId: "your_sender_id",
  appId: "your_app_id",
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);

getToken(messaging, { vapidKey: "your_vapid_key" }).then((currentToken) => {
  if (currentToken) {
    // Send token to backend
  }
});

onMessage(messaging, (payload) => {
  console.log('Message received. ', payload);
});
```

---

## 📬 Sending Test Notification from Firebase Console

You can also test sending notifications from the **Firebase Console** > **Cloud Messaging** section.

---

## 🧪 Testing

- Make sure device token is correct and not expired
- Check browser/OS permissions for notifications
- Check Laravel logs for any API errors from Firebase

---

## 🧹 Cleanup

- Revoke unused device tokens
- Securely store your Firebase Server Key

---

## 📖 References

- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [Laravel HTTP Client](https://laravel.com/docs/http-client)

---
