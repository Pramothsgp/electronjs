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

