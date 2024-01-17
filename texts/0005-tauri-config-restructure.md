- Feature Name: tauri_config_restructure
- Start Date: 2023-13-20
- RFC PR: [tauri-apps/rfcs#13](https://github.com/tauri-apps/rfcs/pull/13)
- Tracking Issue: 

# Summary

Restructuring the `tauri.conf.json` for more simplicity, consistency and clarity.

# Motivation

The current `tauri.conf.json` has a bunch of objects that don't necessarily follow a rule that could reason why they exist there and thus makes it awkward to figure out where to add new fields
for example `tauri > bundle` object is a configuration for the CLI, why does it exist inside `tauri` object, another example is the `plugins` object which is a tauri feature, why does it exist outside of `tauri` object? 
These inconsistencies will make it harder to add new fields to the config. 

# Guide-level explanation

- Unpack `tauri` object to the root object.
- Unpack `package` fields to the root object.
- Move `tauri > bundle > identifier` to the root object as it is used by other places in tauri other than bundling.
- Remove `bundle > updater` as the only useful key it has, is `pubkey` field and that should be moveed to `plugin > updater > pubkey`.
  This will require changes in the CLI to only sign the bundles when `TAURI_SIGNING_PRIVATE_KEY` key is set .
  This will also fix a huge DX when using `tauri-plugin-updater` and you endup having to configure updater-related configs once in `bundle > updater` and `plugin > updater`.
- Move `build > withGlobalTauri` to the root object.
- Move `tauri > pattern` to `security > pattern` and make it accept a simple string for `brownfield`.
- Move `tauri > cli` and `tauri > update` fields to `plugins > cli` and `plugins > update` as they are plugins now.
- Rename `build > distDir` to `frontendDist` to explicitly set the intent of the option.
- Rename `build > devPath` to `build > devUrl` and only accept urls,
  if users don't have a devServer, they should remove this field and only set `build > frontendDist` which will make the CLI
  start its built-in devServer or fallback to embed the assets if `--no-dev-server` is used.

<table>
<thead>
  <tr>
    <th>Current Config</th>
    <th>New Config</th>
  </tr>
</thead>
<tbody>
  <tr>
  <td>

```jsonc
{
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build",
    "devPath": "http://localhost:8080/",
    "distDir": "../src",
    "withGlobalTauri": true
  },
  "package": {
    "productName": "tauri-app",
    "version": "0.0.0"
  },
  "tauri": {
    "allowlist": {
      "all": true
    },
    "bundle": {
      "active": true,
      "targets": "all",
      "identifier": "com.taasduri.dev",
      "icon": [
        "icons/32x32.png",
        "icons/128x128.png",
        "icons/128x128@2x.png",
        "icons/icon.icns",
        "icons/icon.ico"
      ]
    },
    "updater": {
      "active": true,
      "pubkey": "",
      "endpoints": ["http://localhost:8080/update.json"],
      "windows": {
        "installMode": "basicUi"
      }
    },
    "cli": {},
    "macOSPrivateApi": false,
    "security": {
      "csp": null
    },
    "pattern": {
      "use": "brownfield"
    },
    "systemTray": {
      "iconPath": "./path/to/icon"
    },
    "windows": [
      {
        "title": "tauri-app",
        "width": 800,
        "height": 600
      }
    ]
  }
}
```

</td>
<td>

```jsonc
{
  "productName": "tauri-app",
  "version": "0.1.0",
  "identifier": "com.tauri.dev",
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build",
    "devUrl": "http://localhost:8080/",
    "frontendDist": "../src" // or ["../src/index.html", "../src/main.js"] if need to include specific files
  },
  "withGlobalTauri": true,
  "macosPrivateApi": false,
  "windows": [
    {
      "title": "tauri-app",
      "width": 800,
      "height": 600
    }
  ],
  "trayIcon": {
    "iconPath": "./path/to/icon"
  },
  "allowlist": { // soon to be replaced with permissions and cababilities
    "all": true
  },
  "security": {
    "pattern": "brownfield",
    "csp": null
  },
  "plugins": {
    "cli": {},
    "updater": {
      "pubkey": "",
      "endpoints": ["http://localhost:8080/update.json"],
      "windows": {
        "installMode": "basicUi"
      }
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

</td>
  </tr>
</tbody>
</table>

## Community Considerations

This will be a breaking change and so users will need to restructure the config themselves or rely on `tauri migrate` command.

# Drawbacks

Almost all current Tauri users will need to migrate to the new breaking changes but it is better to do this now in a major release.
