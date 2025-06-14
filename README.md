Here’s a **step-by-step guide to initialize a Vite + React + TypeScript + Electron project** from scratch, including **necessary configuration changes** for Electron, TypeScript, and `electron-builder`.

---

## 🛠️ Step-by-Step Project Initialization Guide

---

### 🔁 1. Initialize the Project

```bash
npm create vite@latest myapp --template react-ts
cd myapp
npm install
```

---

### ⚙️ 2. Install Electron and Required Tools

```bash
npm install --save-dev electron electron-builder @types/electron
```

Also install TypeScript and Vite types if not already present:

```bash
npm install --save-dev typescript vite
```

---

### 🧾 3. Create Directory Structure

```bash
mkdir -p src/electron src/ui/
```


Move React app files into `src/ui/` if preferred, or keep as-is.

---

### 🧠 4. Add Electron Entry Point

Create `src/electron/main.ts`:

```ts
import {app , BrowserWindow, ipcMain} from 'electron';
import path from 'path';

type test = string;

app.on('ready', () => {
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false
    }
  });


  mainWindow.loadFile(path.join(app.getAppPath(), '/dist-react/index.html'));
});
```

---

### 🧪 5. Add TypeScript Config Files

#### `tsconfig.json`

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

#### `tsconfig.app.json` (for Vite/React)
Add `"exclude": ["src/electron"]` and create a seperate  tsconfig for electron
```json
{
  "compilerOptions": {
    ...
  },
  "include": ["src"],
  "exclude": ["src/electron"]
}
```

#### `src/electron/tsconfig.json` (for Electron)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "NodeNext",
    "outDir": "../../dist-electron", // output- directory
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["./**/*.ts"]
}
```

---

### 🔧 6. Update `package.json`

```json
{
  "name": "cpu-manager",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "main": "dist-electron/main.js", // electron app entry point
  "scripts": {
    "dev:react": "vite", // running react
    "dev:electron": "electron .", // running electron
    // to skip sandboxing
    //"dev:electron: : 'ELECTRON_DISABLE_SANDBOX=true electron .',
    "build": "tsc -b && vite build",
    "transpile:electron": "tsc -p src/electron/tsconfig.json", // converting ts to js

    // electron build commands
    "dist:mac": "npm run transpile:electron && npm run build && electron-builder --mac --arm64",
    "dist:linux": "npm run transpile:electron && npm run build && electron-builder --linux --x64",
    "dist:win": "npm run transpile:electron && npm run build && electron-builder --win --x64",
    "dist": "npm run dist:mac && npm run dist:linux && npm run dist:win"
  },
  "devDependencies": {
    ...
  },
  "dependencies": {
    ...
  }
}
```

---

### 📦 7. Add `electron-builder.json`

```json
{
  "appId": "com.jarvis-tech.cpu-manager",
  "productName": "CPU Manager",
  "files": [
    "dist-electron",
    "dist-react"
  ],
  "icon": "src/ui/assets/icon.png",
  "mac": {
    "target": "dmg"
  },
  "linux": {
    "target": ["AppImage", "deb"],
    "category": "Utility"
  },
  "win": {
    "target": ["portable", "msi"]
  }
}
```

---

### 🌐 8. Update `vite.config.ts`

Ensure `build.outDir` is set correctly to `dist-react`:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  base: './',
  build: {
    outDir: 'dist-react',
    emptyOutDir: true
  }
});
```

---

### ✅ 9. Add `.gitignore`

```gitignore
node_modules
dist
dist-react
dist-electron
*.log
.vscode/
```

---

### 🚀 10. Run the App

```bash
# Start Vite dev server
npm run dev:react

# Start Electron
npm run dev:electron
```

---

### 📦 11. Build and Package

```bash
# Build Electron + React
npm run build

# Package for platform
npm run dist:win       # On Windows
npm run dist:linux     # On Linux
npm run dist:mac       # On macOS
```

---


Here’s a **cleaned-up and complete development setup guide** for your **Electron + Vite + React + TypeScript** project — including the necessary steps, config, optional tools, and fixes:

---

# ⚙️ Electron + Vite + React – Development Setup Guide

## 📁 Project Structure

```
cpu-manager/
├── src/
│   ├── electron/        # Electron main & preload scripts
│   └── ui/              # React source code
├── dist-react/          # React build output (from Vite)
├── dist-electron/       # Electron transpiled output (from TSC)
├── package.json
├── tsconfig.json
├── electron-builder.json
├── vite.config.ts
└── .gitignore
```

---

## 🧱 Step-by-Step Setup

### 1. Install Dependencies

```bash
npm install
```

---

### 2. Install Dev Tools

#### Cross-env (for Windows/Linux env compatibility)

```bash
npm i --save-dev cross-env
```

#### Optional: Run multiple scripts in parallel

```bash
npm i --save-dev npm-run-all
```

---

### 3. Add `NODE_ENV` & Sandboxing Fix

To expose dev mode & avoid issues with Electron sandboxing:

```ts
// src/electron/utils/util.ts
export function isDev(): boolean {
  return process.env.NODE_ENV === 'development';
}
```

---

### 4. Configure Vite Dev Server

```ts
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    strictPort: true,
  }
});
```

---

### 5. Preload Script (CommonJS)

```js
// src/electron/preload.js (will compile to preload.cjs)

const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('electron', {
  subscribeStatistics: (callback) => callback({}),
  getStatistics: () => console.log('getStatistics called'),
});
```

Make sure it gets compiled to `dist-electron/preload.cjs` by your Electron `tsconfig`.

---

### 6. Preload Path Resolver

```ts
// src/electron/utils/pathresolver.ts
import { app } from "electron";
import path from "path";
import { isDev } from "./util.js";

export const getPreLoadPath = (): string => {
  return path.resolve(
    app.getAppPath(),
    'dist-electron',
    'preload.cjs'
  );
};
```

---

### 7. Electron Main Process

```ts
// src/electron/main.ts
import { app, BrowserWindow } from 'electron';
import path from 'path';
import { isDev } from './utils/util.js';
import { getPreLoadPath } from './utils/pathresolver.js';
import { pollresource } from './resourceManager.js';

app.on('ready', () => {
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: getPreLoadPath(),
      contextIsolation: true,
      nodeIntegration: false,
    },
  });

  pollresource();

  if (isDev()) {
    mainWindow.loadURL('http://localhost:5173');
  } else {
    mainWindow.loadFile(path.join(app.getAppPath(), 'dist-react/index.html'));
  }
});
```

---

### 8. Update `package.json` Scripts

```json
{
  "scripts": {
    "dev": "npm-run-all --parallel dev:react dev:electron",
    "dev:react": "vite",
    "dev:electron": "npm run transpile:electron && cross-env ELECTRON_DISABLE_SANDBOX=true NODE_ENV=development electron .",
    "transpile:electron": "tsc -p src/electron/tsconfig.json",
    "build": "tsc -b && vite build",
    "dist:mac": "npm run transpile:electron && npm run build && electron-builder --mac --arm64",
    "dist:linux": "npm run transpile:electron && npm run build && electron-builder --linux --x64",
    "dist:win": "npm run transpile:electron && npm run build && electron-builder --win --x64",
    "dist": "npm run dist:mac && npm run dist:linux && npm run dist:win"
  }
}
```

---

## ✅ Development Start

```bash
npm run dev
```

This starts both:

* React dev server on `http://localhost:5173`
* Electron app with live reload using the local server

---

## 🛠 Common Gotchas & Fixes

| Problem                          | Fix                                                                |
| -------------------------------- | ------------------------------------------------------------------ |
| `Cannot find module preload.cjs` | Ensure the file is built and path is correct in `getPreLoadPath()` |
| `sandbox` errors                 | Set `ELECTRON_DISABLE_SANDBOX=true`                                |
| `NODE_ENV` is undefined          | Use `cross-env` to set it consistently                             |
| Type issues with CJS preload     | Use `.cjs` and CommonJS syntax (`require`, not `import`)           |

---

