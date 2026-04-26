# ELECTRONJS-POLICY.md

## Purpose

Define standards for building secure, maintainable, and well-packaged Electron.js desktop applications. This policy governs agent behavior when generating Electron main process, renderer process, IPC, and packaging code.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Electron main process code (`main.js` / `main.ts`).
- Renderer process code and preload scripts.
- IPC communication between processes.
- App packaging configuration (electron-builder, electron-forge).
- Auto-update integration.
- Desktop-specific UX patterns.

---

## Core Principles

- **Security first.** The renderer is untrusted by default. Treat it like a web browser.
- **Process separation.** Main process orchestrates; renderer process renders UI. No direct Node.js access from the renderer.
- **IPC is the only bridge.** All communication between main and renderer goes through typed IPC channels.
- **Minimal permissions.** Request only the OS capabilities the app requires.
- **Packaging is production configuration.** Build and auto-update setup is part of the codebase, not an afterthought.

---

## Required Behavior

### Security Configuration
- Always set `contextIsolation: true` in `BrowserWindow` `webPreferences`.
- Always set `nodeIntegration: false` in `webPreferences`.
- Never set `webSecurity: false` unless serving local files with a custom protocol and the reason is documented.
- Set a strict Content Security Policy (CSP) in the renderer HTML's `<meta>` tag or via HTTP headers.
- Load only trusted, verified URLs in `BrowserWindow` — validate URLs before loading remote content.

```js
const mainWindow = new BrowserWindow({
  webPreferences: {
    contextIsolation: true,
    nodeIntegration: false,
    preload: path.join(__dirname, 'preload.js'),
    sandbox: true,
  },
});
```

### Preload Scripts
- All Node.js APIs exposed to the renderer must go through the preload script.
- Use `contextBridge.exposeInMainWorld()` to expose a controlled, typed API surface.
- Expose only the minimum set of capabilities required.
- Validate all data received from the renderer in the preload or main process before acting on it.

```js
// preload.js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('api', {
  openFile: () => ipcRenderer.invoke('dialog:openFile'),
  onUpdate: (callback) => ipcRenderer.on('update:available', (_event, info) => callback(info)),
});
```

### IPC Hygiene
- Use `ipcMain.handle` (invoke/handle pattern) for request/response IPC — not `ipcMain.on` with manual `event.reply`.
- Define all IPC channel names as constants in a shared module (`ipc-channels.ts`).
- Validate and sanitize all data received from the renderer via IPC before processing.
- Do not pass `event` objects or `webContents` references to renderer processes.
- Limit IPC to what the renderer legitimately needs — do not expose arbitrary file system or shell access.

```ts
// ipc-channels.ts
export const IPC = {
  OPEN_FILE: 'dialog:openFile',
  SAVE_FILE: 'dialog:saveFile',
} as const;

// main process
ipcMain.handle(IPC.OPEN_FILE, async () => {
  const result = await dialog.showOpenDialog({ properties: ['openFile'] });
  return result.filePaths[0] ?? null;
});
```

### Main Process Responsibilities
- Create and manage `BrowserWindow` instances.
- Handle OS integration: tray, menus, notifications, file associations.
- Perform file I/O, native dialogs, and system calls.
- Manage the auto-update lifecycle.
- Do not place UI logic or rendering concern in the main process.

### Renderer Process Responsibilities
- Render UI using the chosen framework (React, Vue, vanilla JS).
- Communicate with the main process exclusively through the exposed `contextBridge` API.
- Treat the main process API as an async boundary — always use `await` on `invoke` calls.
- Do not require or import Node.js modules directly in renderer code.

### Auto-Updates
- Use `electron-updater` (with electron-builder) for auto-update support.
- Check for updates on app start and provide user-visible progress and confirmation before installing.
- Handle update errors gracefully — do not crash the app if the update server is unreachable.
- Sign the application binary; unsigned auto-updates will be blocked on macOS and Windows.

### Packaging
- Use `electron-builder` or `electron-forge` for packaging and distribution.
- Configure `productName`, `appId`, `copyright`, and icon assets in the build config.
- Enable code signing for macOS (notarization required) and Windows (Authenticode signing).
- Pin the Electron version in `package.json` — do not use `latest` or `*` for production.
- Exclude development dependencies and source maps from the packaged app.

### Desktop UX
- Match the native platform's interaction patterns where possible (window controls, menus, keyboard shortcuts).
- Use `app.setName()` and `app.setAboutPanelOptions()` for proper OS integration.
- Handle `app.requestSingleInstanceLock()` to prevent duplicate instances.
- Persist window size and position across sessions using `electron-store` or equivalent.

---

## Things to Avoid

- Setting `nodeIntegration: true` in any `BrowserWindow`.
- Setting `contextIsolation: false`.
- Exposing `require`, `shell`, or arbitrary Node APIs directly via `contextBridge`.
- Using `shell.openExternal()` with unvalidated URLs (open redirect / phishing risk).
- Executing child processes with user-supplied input without sanitization.
- Loading remote content in `BrowserWindow` without CSP and URL validation.
- Using `ipcMain.on` with manual reply instead of `ipcMain.handle`.
- Committing code-signing certificates or private keys to source control.

---

## Output Expectations

Generated Electron code must:
- Set `contextIsolation: true` and `nodeIntegration: false` on all `BrowserWindow` instances.
- Use preload scripts with `contextBridge.exposeInMainWorld()` for all renderer/main communication.
- Define IPC channels as named constants.
- Use `ipcMain.handle` for all request/response IPC.
- Include packaging configuration with `productName`, `appId`, and code-signing stubs.
- Not expose Node.js APIs directly to the renderer.

---

## Optional Checklist

- [ ] `contextIsolation: true` on all `BrowserWindow` instances.
- [ ] `nodeIntegration: false` on all `BrowserWindow` instances.
- [ ] Preload script uses `contextBridge.exposeInMainWorld()`.
- [ ] IPC channels defined as constants.
- [ ] `ipcMain.handle` used for request/response IPC.
- [ ] All renderer input validated in main process before use.
- [ ] `shell.openExternal()` only called with validated URLs.
- [ ] Auto-update configured with user confirmation.
- [ ] Packaging config includes `appId`, `productName`, and signing stubs.
- [ ] `app.requestSingleInstanceLock()` implemented.
