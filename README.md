# Configure registry for Yarn
#### Yarn config get registry
Set mirror registry:
```sh 
yarn config set registry https://registry.npm.taobao.org
```
Set original registry:
```sh 
yarn config set registry https://registry.yarnpkg.com
```
#### Network connection error
If the following message appears during package installation, it means the proxy cannot connect to remote server:
```sh
info There appears to be trouble with your network connection. Retrying...
```
Reset the proxy will solve the problem:
```sh
yarn config delete proxy
npm config rm proxy
npm config rm https-proxy
```

# Install packages
### Install packages with proxy
The following methods are tested and proves to be working.

To install all packages:
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn
```
To install certain packages:
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn add <package names>
```

The following methods may work, but need farther testing:
```sh
ELECTRON_MIRROR="https://cdn.npm.taobao.org/dist/electron/""  yarn add <<package names>
```
```sh
yarn add <package names> --proxy <local proxy IP address> (http://127.0.0.1:3561)
```
```sh
yarn add <package names> --ignore-engines --registry=https://registry.npm.taobao.org 
```

### Electron module failed to install correctly
If the following problem is encountered during *"yarn add electron"* or *"yarn start"*, that means it's unable to download from electron dist.
```sh
[Error] 
Electron failed to install correctly, please delete node_modules/electron and try installing again
```
To solve the problem, delete "node_modules/electron" folder, and use TAOBAO mirror to download electron module again.
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn add electron
```

# Electron Rebuild 
### Integrate @serialport module
If the following error is encountered during the rebuild stage, that means @serialport module is not installed properly, most likely due to the  ***Greate Wall***.
```sh
[Error] 
An unhandled error occurred inside electron-rebuild
argv.t.split is not a function

TypeError: argv.t.split is not a function
```
```sh
[Error] 
... cannot find serialport ...
```
To solve the above problem, follow these steps:
1. Remove old serialport module from "app/node_modules";
2. Add new serialport module to "app/node_modules";
3. cd into "app" folder, and run electron rebuild command, do replace the "--proxy http://127.0.0.1:3561" with your own proxy setting:
    ```sh
    ../node_modules/.bin/electron-rebuild --proxy http://127.0.0.1:3561 -dist-url=https://npm.taobao.org/mirrors/atom-shell
    ```
4. Copy corresponding version of [serialport bindings](https://github.com/serialport/node-serialport/tags) into "app\node_modules\@serialport\bindings\build\Release" folder. Check which version of binding to use with the following table.

| Node version  | serialport binding |
| ------------- |:------------------:|
| v10.xx.x      | v57                |
| v12.xx.x      | v75                |

**NOTE:** If *"app/node_modules/@serialport"* already exists, skip step 1 & 2.


### gyp-rebuild (with mirror)
Run the following command if gyp-rebuild is needed, do change the "target" to corresponding electron version number.
```sh
node-gyp rebuild --target=7.1.13 --arch=x64 --dist-url=https://npm.taobao.org/mirrors/atom-shell --msvs_version=2017
--dist-url=https://electronjs.org/headers"
```

# Electron Package 
Run the following command
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn package
```

### Stuck at packing stage
Packaging stucks, console displays *"downloading  url=https://github.com/electron/electron/releases/download/[version]//electron-[version]-win32-x64.zip"*;
This problem is caused by the **Greate Wall**. To solve this problem, download the zip file by copying the url directly into the broswer. 
Put the downloaded zip file into *"C:\Users\<username>\AppData\Local\electron\Cache"*, then run the packaging again.

### Connection problem during yarn package
1. Check the error message, find out what are the missing resources; (most probably "nsis" "winCodeSign")
2. Download the [missing binaries](https://github.com/electron-userland/electron-builder-binaries), and put into "C:\Users\UserName\AppData\Local\electron-builder\Cache"

### Out of memroy problem
If the following error appears during the package step, that means the memory limit is reached.
```sh
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```
Do the following to solve the problem:
1. Move dependency lib to app.html <script> tags;
2. Add memory size flag to yarn package command, such as 
```sh 
yarn package --max_old_space_size=4096
```

### Unable to load preload script in packaged app
Modify the */app/main.dev.ts* file from this:

```Javascript
mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728,
    fullscreen: true,
    webPreferences:
      process.env.NODE_ENV === 'development' || process.env.E2E_BUILD === 'true'
        ? {
            nodeIntegration: true
          }
        : {
            preload: path.join(__dirname, 'dist/renderer.prod.js')
          }
  });
```

to this:

```Javascript
mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728,
    fullscreen: true,
    webPreferences: {
      nodeIntegration: true
    }
  });
```

### MSBuild error
If the error **"MSBuild.exe ENOENT"** is encountered, that means the MSBuild tool is not properly set, do the following:
1. Download and install [Visual C++ Build tools](https://go.microsoft.com/fwlink/?LinkId=691126);
2. Configure npm to use python 2.7 and VS2015:
```sh
npm config set python python2.7
npm config set msvs_version 2015
```

If the error **"node.lib fatal error LNK1106"** is encountered, that means node.lib is not downloaded properly. It needs to be manully downloaded.
1. Download the corresponding version of node.lib from [here](https://nodejs.org/download/release/) (https://nodejs.org/download/release/[version]/win-x64/)
2. Copy to replace the node.lib in your local folder:
```
C:\Users\<username>\AppData\Local\node-gyp\Cache\[version]\x64
``` 

# Example: Upgrade Electron-React App
### Install packages
1. Git clone electron-react-boilplate into local folder;
2. Yarn install packages with the following command:
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn
```
3. Yarn install packages for App:
A faster way is to install at one go, the following command is to replace step2 to step 5.
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn add @material-ui/core @material-ui/icons @material-ui/lab @material-ui/styles material-table eventproxy immer lodash nedb dom-to-image js-file-download mdi-material-ui react-awesome-button recharts serialport
```

4. cd into */app* folder, run the following command to install serialport as native module for the app:
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn add serialport
```

Then download the corresponding version of [serialport bindings](https://github.com/serialport/node-serialport/tags) into *"\app\node_modules\@serialport\build\Release\bindings.node"*.
Refer to the following table for which version of binding node to use.

| ER boilerplate| electron           | serialport binding   | status   |
| ------------- |:------------------:|:--------------------:|:--------:|
| v1.1.0        | v7.1.13            | v75                  | OK       |
| v1.2.0        | v9.0.4             |                      |          |
| v1.3.1        | v8.5.0             | v76                  | OK       |
| v1.4.0        | v11.0.3            |                      | Not supported |

5. Do the rest of migration, follow the steps in the following session.

### Migration of old App
1. Add the following folders and files: 

| Folder/Files  | Comments           |
| ------------- |:------------------:|
| /database     | app database       |
| /worker       | local worker libs  |
| /my_resources | third party libs   |
| /configs/nedbHotfixForElectron.js  |   neDB hot fix plugin  |

2. Modify the following files:

| Files         | Comments           |
| ------------- |:------------------:|
| /tsconfig.json| include & exclude paths|
| /gitignore    | ignore paths           |
| /config/webpack.config.base.js | neDBFix; Module Alias;|
| /app/main.dev.ts               | set nodeIntegrationInWorker, enableRemoteModule |
| /app/app.html                  | include libs |
| /app/app.global.css            | change global styles|

3. Remove the following folder and files:

| Folder/Files  | Comments           |
| ------------- |:------------------:|
| /components   |                    |
| /constants    |                    |
| /containers   |                    |
| /features     |                    |
| index.tsx     |                    |
| menu.ts       |                    |
| rootReducer.ts|                    |
| Routes        |                    |
| store         |                    |

4. Copy files from old app folder into the new one:

| Folder/Files  | Comments           |
| ------------- |:------------------:|
| /actions      |                    |
| /components   |                    |
| /configuration |                    |
| /constants     |                    |
| /containers    |                    |
| /drivers       |                    |
| /features      |                    |
| /interfaces    |                    |
| /middlewares   |                    |
| /reducers      |                    |
| /routes        |                    |
| /store         |                    |
| /tools         |                    |
| /index.tsx     |                    |
| /menu.ts       |                    |

# Other development issues

### git clone too slow
Git clone too slow is usually caused by the ***Great Wall***. To solve this issue, git proxy needs to be set.
Run the following commands before clone:

```sh
git config --global https.proxy <local proxy IP address> (http://127.0.0.1:3561)
git config --global http.proxy <local proxy IP address> (http://127.0.0.1:3561)
```

To unset the proxy, run the following commands:
```sh
git config --global --unset https.proxy
git config --global --unset http.proxy
```

### Could not detect node-abi error
This error usually happens after a new electron version is installed.
Run the following command:
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn upgrade electron-rebuild
```

### Use NVM (node version manager) to manage node version
1. Download and install [nvm-windows](https://github.com/coreybutler/nvm-windows/releases/tag/1.1.7) from github;
2. Do not install node from nvm command line; download node zip from [this link](https://nodejs.org/en/download/releases/);
3. Copy and unzip the downloaded zip file into (C:\Users\UserName\AppData\Roaming\nvm); rename the folder to corresponding version (12.20.0 etc.);
4. Use nvm to control the node version.

### Electron worker "require is not defined" problem
Add "nodeIntegrationInWorker:true" in webPreferences. (Inside menu.ts)

### Missing Material-table Icons
Add the following to ***app.html***
```html
<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
```

### neDB will automatic use broswer DB
Use fixNedbForElectron plugin for webpack to fix it.

### Fix Platform IO wall problem
- Use LANTERN PRO, enable "Proxy all traffic" in settings;
- Enable VPN in Windows Internet Options / Connection