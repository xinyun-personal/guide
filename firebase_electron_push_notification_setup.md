# Setting Up Firebase Push Notifications in Electron

This guide will walk you through the steps to set up Firebase Cloud Messaging (FCM) in an Electron app, enabling push notifications both in the foreground and background.

## Step 1: Set up Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/).
2. Create a new project or use an existing one.
3. Go to **Project Settings** and under **General**, add a new Web App to the project.
4. Copy the Firebase configuration object provided (API Key, Project ID, etc.).
5. Enable Firebase Cloud Messaging (FCM) by navigating to the **Cloud Messaging** tab.

## Step 2: Install Firebase SDK

1. In your Electron project folder, install Firebase SDK:
    ```bash
    npm install firebase
    ```

## Step 3: Initialize Firebase in Electron

1. Create a new file named `firebase.js` in the `src` folder.
2. In `firebase.js`, initialize Firebase using the configuration from Firebase Console and set up the messaging functions.

### `firebase.js`

```js
const {initializeApp} = require('firebase/app');
const {getMessaging, getToken, onMessage, onBackgroundMessage} = require('firebase/messaging');

const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_STORAGE_BUCKET",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID",
    measurementId: "YOUR_MEASUREMENT_ID",
};

const appInstance = initializeApp(firebaseConfig);
const messaging = getMessaging(appInstance);

// Request Notification Permission
async function requestNotificationPermission() {
    try {
        const token = await getToken(messaging, {vapidKey: 'YOUR_VAPID_KEY'});
        console.log('FCM Token:', token);
        return token;
    } catch (error) {
        console.error('Error getting token', error);
    }
}

// Handle Foreground Notifications
onMessage(messaging, (payload) => {
    console.log('Message received in foreground: ', payload);
    new Notification({
        title: payload.notification.title,
        body: payload.notification.body
    }).show();
});

// Handle Background Notifications
onBackgroundMessage(messaging, (payload) => {
    console.log('Received background message: ', payload);
    const notification = new Notification({
        title: payload.notification.title,
        body: payload.notification.body,
    });
    notification.show();
});

module.exports = {requestNotificationPermission};

```

## Step 4: Update Main Process in Electron

1. In the src/main.js file, import firebase.js and set up the call to request notification permission when the app starts.

## `electron/main.js`

```js
const {app, BrowserWindow, Menu} = require('electron');
const {requestNotificationPermission} = require('./firebase'); // Import Firebase setup

app.on('ready', () => {
    const mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            preload: path.join(__dirname, './preload.js'),
            nodeIntegration: false,
            contextIsolation: true,
        },
    });

    mainWindow.loadFile(path.join(__dirname, '../dist/index.html'));
    requestNotificationPermission();
});
```

## Step 5: Expose Firebase API to Renderer Process

1. Create a preload.js file to expose the necessary Firebase methods to the renderer process.

## `electron/preload.js`

```js
const {contextBridge, ipcRenderer} = require("electron");

contextBridge.exposeInMainWorld("electron", {
    requestNotificationPermission: () => ipcRenderer.invoke('requestNotificationPermission')
});
```

## Step 6: Handle Push Notification in Vue.js

1. In your Vue component, create a button or UI to allow the user to enable push notifications.

```vue

<template>
  <div>
    <h1>Push Notifications</h1>
    <button @click="enableNotifications()">Enable Notifications</button>
  </div>
</template>

<script>
  export default {
    methods: {
      enableNotifications() {
        window.top.electron.requestNotificationPermission();
      },
    },
  };
</script>
```

## Step 7: Set Up Service Worker for Background Notifications

1. For background notifications to work, Electron needs a service worker.
2. Create firebase-messaging-sw.js in your public folder.

## `firebase-messaging-sw.js`

```js
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging.js');

const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_STORAGE_BUCKET",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID",
    measurementId: "YOUR_MEASUREMENT_ID",
};

firebase.initializeApp(firebaseConfig);
const messaging = firebase.messaging();

messaging.onBackgroundMessage(function (payload) {
    const notificationTitle = payload.notification.title;
    const notificationOptions = {
        body: payload.notification.body,
    };

    self.registration.showNotification(notificationTitle, notificationOptions);
});

```

## Step 8: Register Service Worker in Vue Component

## `src/main.js` (Renderer Process):

```js
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/firebase-messaging-sw.js')
        .then(function (registration) {
            console.log('Service Worker registered with scope:', registration.scope);
        })
        .catch(function (error) {
            console.log('Service Worker registration failed:', error);
        });
}
```