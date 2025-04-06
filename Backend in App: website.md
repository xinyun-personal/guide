
# 📦 Fullstack Cross-Platform App – Step-by-Step Overview

Build a fullstack application that works seamlessly on Android, iOS, and Desktop using modern web technologies.

---

## 🧱 Architecture Overview

| Layer      | Tech Stack                                  |
|------------|----------------------------------------------|
| Backend    | Node.js + Hono.js (built with Webpack)       |
| Frontend   | Vue.js + Vite                                |
| Mobile     | Capacitor + nodejs-mobile-react              |
| Desktop    | Electron (bundled frontend + backend)        |

---

## 🔧 Step 1: Setup Backend (Node.js + Hono.js + Webpack)

### 1.1 Initialize Backend

```bash
mkdir backend && cd backend
npm init -y
npm install hono
npm install --save-dev webpack webpack-cli ts-loader typescript
```

### 1.2 Create Basic `Hono` Server (`index.js`)

```js
import { Hono } from 'hono';

const app = new Hono();

app.get('/api/hello', (c) => c.json({ message: 'Hello from backend!' }));

// Access the app server
const server = app.server;

// Export the server for use in mobile environments
export default server;
```

### 1.3 Webpack Configuration (`webpack.config.js`)

```js
const path = require('path');

module.exports = {
  entry: './index.ts',
  target: 'node',
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  },
  output: {
    filename: 'server.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

### 1.4 Compile Backend

```bash

npx webpack --config webpack.config.js
```

---

## 🎨 Step 2: Setup Frontend (Vue.js + Vite)

### 2.1 Create Frontend App

```bash

cd ..
npm create vite@latest frontend -- --template vue
cd frontend
npm install
```

### 2.2 Example API Call from Vue

```js
// frontend/src/api.ts
export async function getHello() {
  const res = await fetch('http://localhost:3000/api/hello');
  return res.json();
}
```

### 2.3 Use in Component

```vue

<script setup>
import { onMounted, ref } from 'vue';
import { getHello } from './api';

const message = ref('');

onMounted(async () => {
  const data = await getHello();
  message.value = data.message;
});
</script>

<template>
  <div>{{ message }}</div>
</template>
```

---

## 📱 Step 3: Mobile App (Capacitor + nodejs-mobile-react)

### 3.1 Initialize Capacitor

```bash

npm install --global @capacitor/cli
cd frontend
npx cap init my-app com.example.app
npx cap add android
npx cap add ios
```

### 3.2 Add nodejs-mobile-react Backend

1. Install `nodejs-mobile-react` native module in `android` and `ios`.
2. Copy `backend/dist/server.js` into Capacitor assets folder.
3. Start the Node.js backend inside your Android/iOS code (Java/Swift).
4. Access it from Vue via `http://localhost:3000`.

> **Note:** You'll need to bridge the native layer to run Node.js backend code.
```bash
#install library in
npm install nodejs-mobile-react-native
``` 

```java

include ':nodejs-mobile-react-native'
project(':nodejs-mobile-react-native').projectDir = new File(rootProject.projectDir, '../node_modules/nodejs-mobile-react-native/android')
```


### 3.2.1 Prepare Node.js Backend for Mobile

1. Compile your backend with Webpack (`server.js`).
2. Copy it into your mobile assets:

   - **Android**:  
     `android/app/src/main/assets/nodejs-project/server.js`

   - **iOS**:  
     Add `server.js` to your Xcode project's resources or `Copy Bundle Resources`.

3. Also copy the necessary `node_modules` (like `hono`, `http`, etc.) into that folder.

---

### 3.2.2 Android – Start Node.js in `MainActivity.java`

```java
import org.nodejs.mobile.NodeJS;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Start the Node.js engine
    NodeJS.start(getApplicationContext());

    // Run the server.js script located in assets/nodejs-project
    NodeJS.runScript("server.js", getAssets());

    // Load your Capacitor web app
    load();
}
```

### 3.2.3 iOS – Start Node.js in AppDelegate.swift

```swift
import NodejsMobile

func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    // Start Node.js engine
    NodeRunner.startEngine()

    // Run the server script
    NodeRunner.runScript("server.js")

    return true
}
```

### 3.4 Sync Frontend with Capacitor

```bash
npx cap copy
npx cap open android  # or ios
```

---

## 💻 Step 4: Desktop App (Electron + Vue + Node)

### 4.1 Install Electron in Project Root

```bash
npm install electron --save-dev
```

### 4.2 Create Electron Entry (`main.js`)

```js
// desktop/main.js
const { app, BrowserWindow } = require('electron');
const path = require('path');
const { spawn } = require('child_process');

function createWindow() {
  const win = new BrowserWindow({ width: 800, height: 600 });
  win.loadFile('dist/index.html');

  // Start backend
  const server = spawn('node', [path.join(__dirname, '../backend/dist/server.js')]);
}

app.whenReady().then(createWindow);
```

### 4.3 Build Vue Frontend for Production

```bash
cd frontend
npm run build
```

### 4.4 Run Electron App

```bash
cd desktop
npx electron main.js
```



---

## 📁 Suggested Folder Structure

```
project-root/
├── backend/               # Node.js + Hono.js API
│   ├── index.ts
│   ├── webpack.config.js
│   └── dist/server.js
├── frontend/              # Vue.js frontend (Vite)
│   └── src/
├── mobile/                # Capacitor native wrapper
│   ├── android/
│   ├── ios/
│   └── node-backend/      # Node.js assets for mobile
├── desktop/               # Electron app
│   └── main.js
├── shared/                # Shared types / utilities (optional)
```

---

## 🔄 Communication Between Frontend & Backend

| Platform    | Frontend             | Backend                  | Communication Method      |
|-------------|----------------------|---------------------------|----------------------------|
| Web/Desktop | Vue (Vite + Electron)| Node.js (Electron)        | HTTP / IPC                 |
| Mobile      | Vue (Capacitor)      | Node.js (nodejs-mobile)   | `http://localhost:3000`    |

---

## ✅ Checklist Summary

- [x] Node.js + Hono.js backend bundled with Webpack
- [x] Vue.js + Vite frontend with API integration
- [x] Mobile support using Capacitor + nodejs-mobile-react
- [x] Desktop support using Electron
- [x] Unified backend logic across all platforms
- [x] Clean project structure and communication flow

---

## 📚 Resources

- [Hono.js](https://hono.dev/)
- [Vue.js](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [Capacitor](https://capacitorjs.com/)
- [nodejs-mobile](https://github.com/JaneaSystems/nodejs-mobile-react)
- [Electron](https://www.electronjs.org/)
