# Vite + Electron template

```
pnpm create @quick-start/electron
```

Then follow the prompts!

```
✔ Project name: <project-name>
✔ Select a framework: › react
✔ Add TypeScript? Yes
✔ Add Electron updater plugin? No
✔ Enable Electron download mirror proxy? No
```

Now run

```
cd <project-name>
pnpm install
```

# Codebase configuration

#### .vscode/settings.json

```
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "always",
    "source.organizeImports": "always"
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.wordWrap": "on",
  "markdownlint.config": {
    "MD041": false
  }
}
```

#### .eslintrc.cjs

```
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    '@electron-toolkit/eslint-config-ts/recommended',
    '@electron-toolkit/eslint-config-prettier'
  ],
  rules: {
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-unused-vars': 'off'
  }
}
```

#### src/main/index.ts

```
import { electronApp, is, optimizer } from '@electron-toolkit/utils'
import { app, BrowserWindow, ipcMain, shell } from 'electron'
import { join } from 'path'
import icon from '../../resources/icon.png?asset'

function createWindow(): void {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 900,
    height: 670,
    show: false,
    autoHideMenuBar: true,
    ...(process.platform === 'linux' ? { icon } : {}),
    center: true,
    title: 'My Template',
    frame: false,
    vibrancy: 'under-window', // needed for blurred background
    visualEffectState: 'active',
    titleBarStyle: 'hidden',
    trafficLightPosition: { x: 15, y: 10 },
    webPreferences: {
      preload: join(__dirname, '../preload/index.js'),
      sandbox: true,
      contextIsolation: true
    }
  })

  mainWindow.on('ready-to-show', () => {
    mainWindow.show()
  })

  mainWindow.webContents.setWindowOpenHandler((details) => {
    shell.openExternal(details.url)
    return { action: 'deny' }
  })

  // HMR for renderer base on electron-vite cli.
  // Load the remote URL for development or the local html file for production.
  if (is.dev && process.env['ELECTRON_RENDERER_URL']) {
    mainWindow.loadURL(process.env['ELECTRON_RENDERER_URL'])
  } else {
    mainWindow.loadFile(join(__dirname, '../renderer/index.html'))
  }
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(() => {
  // Set app user model id for windows
  electronApp.setAppUserModelId('com.electron')

  // Default open or close DevTools by F12 in development
  // and ignore CommandOrControl + R in production.
  // see https://github.com/alex8088/electron-toolkit/tree/master/packages/utils
  app.on('browser-window-created', (_, window) => {
    optimizer.watchWindowShortcuts(window)
  })

  // IPC test
  ipcMain.on('ping', () => console.log('pong'))

  createWindow()

  app.on('activate', function () {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

// In this file you can include the rest of your app"s specific main process
// code. You can also put them in separate files and require them here.

```

#### src/preload/index.ts

```
import { contextBridge } from 'electron'

if (!process.contextIsolated) {
  throw new Error('contextIsolation must be enabled in the BrowserWindow')
}

try {
  contextBridge.exposeInMainWorld('context', {})
} catch (error) {
  console.error(error)
}
```

#### src/preload/index.d.ts

```
declare global {
  interface Window {
    // electron: ElectronAPI
    context: {}
  }
}
```

#### src/renderer/index.html

```
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta
      http-equiv="Content-Security-Policy"
      content="default-src 'self'; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-eval';"
    />
  </head>

  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

#### \_delete\_ icons in src/renderer/src/assets

#### \_delete\_ src/renderer/src/assets/base.css

#### \_delete\_ lines in src/renderer/src/assets/main.css

#### \_delete\_ src/renderer/components/Version.tsx

#### src/renderer/src/App.tsx

```
function App(): JSX.Element {
  return <div></div>
}

export default App
```

# Make directories

```
~/root
touch src/renderer/src/components/index.ts
mkdir src/renderer/src/hooks
mkdir src/renderer/src/store
mkdir src/renderer/src/utils
touch src/renderer/src/utils/index.ts
mkdir src/shared

mkdir src/main/lib
```

# Update configuration files

#### tsconfig.web.json

```
{
  "extends": "@electron-toolkit/tsconfig/tsconfig.web.json",
  "include": [
    "src/renderer/src/env.d.ts",
    "src/renderer/src/**/*",
    "src/renderer/src/**/*.tsx",
    "src/preload/*.d.ts",
    "src/shared/**/*",
  ],
  "compilerOptions": {
    "composite": true,
    "jsx": "react-jsx",
    "noUnusedLocals": false,
    "baseUrl": ".",
    "paths": {
      "@renderer/*": [
        "src/renderer/src/*"
      ],
      "@shared/*": [
        "src/shared/*"
      ],
      "@/*": [
        "src/renderer/src/*"
      ],
    }
  }
}
```

#### tsconfig.node.json

```
{
  "extends": "@electron-toolkit/tsconfig/tsconfig.node.json",
  "include": [
    "electron.vite.config.*",
    "src/main/**/*",
    "src/preload/*",
    "src/shared/**/*"
  ],
  "compilerOptions": {
    "composite": true,
    "types": [
      "electron-vite/node"
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/main/*"
      ],
      "@shared/*": [
        "src/shared/*"
      ],
    }
  }
}
```

#### electron.vite.config.ts

```
import react from '@vitejs/plugin-react'
import { defineConfig, externalizeDepsPlugin } from 'electron-vite'
import { resolve } from 'path'

export default defineConfig({
  main: {
    plugins: [externalizeDepsPlugin()],
    resolve: {
      alias: {
        '@/lib': resolve('src/main/lib'),
        '@shared': resolve('src/shared')
      }
    }
  },
  preload: {
    plugins: [externalizeDepsPlugin()]
  },
  renderer: {
    assetsInclude: 'src/renderer/assets/**',
    resolve: {
      alias: {
        '@renderer': resolve('src/renderer/src'),
        '@shared': resolve('src/shared'),
        '@/hooks': resolve('src/renderer/src/hooks'),
        '@/assets': resolve('src/renderer/src/assets'),
        '@/store': resolve('src/renderer/src/store'),
        '@/components': resolve('src/renderer/src/components'),
        '@/mocks': resolve('src/renderer/src/mocks')
      }
    },
    plugins: [react()]
  }
})
```

# Set up TailwindCSS

```
pnpm install -D tailwindcss postcss autoprefixer tailwind-merge clsx react-icons
npx tailwindcss init -p
```

#### tailwind.config.js

```
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/renderer/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {}
  },
  plugins: []
}
```

#### src/renderer/src/assets/main.css

```
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  #root {
    @apply h-full;
  }

  html,
  body {
    @apply h-full;

    @apply select-none;

    @apply bg-transparent;

    @apply font-mono antialiased text-white;

    @apply overflow-hidden;
  }
}
```

#### src/renderer/src/utils/index.ts

```
import clsx, { ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export const cn = (...args: ClassValue[]) => {
  return twMerge(clsx(...args))
}

```
