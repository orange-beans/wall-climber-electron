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

The following methods may work, and need farther testing:
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
If the following problem is encountered during "yarn add electron" or "yarn start", that means it is not able to download electron dist.
```sh
Electron failed to install correctly, please delete node_modules/electron and try installing again
```
To solve the problem, delete "node_modules/electron" folder, and do use TAOBAO mirror to download electron module again.
```sh
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/" yarn add electron
```

# Electron Rebuild 
### Integrate @serialport module
If the following error is encountered during the rebuild stage, that means @serialport module is not installed properly, most likely due to the  ***Greate Wall***.
```sh
TypeError: argv.t.split is not a function
```
```sh
... cannot find serialport ...
```
To solve the methioned problem, do the following steps:
1. Remove old serialport module from "app/node_modules";
2. Add new serialport module to "app/node_modules";
3. Run electron rebuild command, do replace the "--proxy http://127.0.0.1:3561" with your own proxy setting:
    ```sh
    ../node_modules/.bin/electron-rebuild --proxy http://127.0.0.1:3561 -dist-url=https://npm.taobao.org/mirrors/atom-shell
    ```
4. Copy corresponding version of [serialport bindings](https://github.com/serialport/node-serialport/tags). into "app\node_modules\@serialport\bindings\build\Release" folder. Check which version of binding to use with the following table.

| Node version  | serialport binding |
| ------------- |:------------------:|
| v10.xx.x      | v57                |
| v12.xx.x      | v75                |


### gyp-rebuild (with mirror)
Run the following command if gyp-rebuild is needed, do change the "target" to corresponding electron version number.
```sh
node-gyp rebuild --target=7.1.13 --arch=x64 --dist-url=https://npm.taobao.org/mirrors/atom-shell --msvs_version=2017
--dist-url=https://electronjs.org/headers"
```

### MSBuild error
If the error "MSBuild.exe ENOENT" is encountered, that means the MSBuild tool is not properly set, do the following:
1. Download and install [Visual C++ Build tools](https://go.microsoft.com/fwlink/?LinkId=691126);
2. Configure npm to use python 2.7 and VS2015:
```sh
npm config set python python2.7
npm config set msvs_version 2015
```

# Electron Package 
### Connection problem during yarn package
1. Check the error message, find out what are the missing resources; (mostly probably "nsis" "winCodeSign")
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

# Other development issues
### Electron worker "require is not defined" problem
Add "nodeIntegrationInWorker:true" in webPreferences.

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


### Use NVM (node version manager) to manage node version
1. Download and install [nvm-windows](https://github.com/coreybutler/nvm-windows/releases/tag/1.1.7) from github;
2. Do not install node from nvm command line; download node zip from [this link](https://nodejs.org/en/download/releases/);
3. Copy and unzip the downloaded zip file into (C:\Users\UserName\AppData\Roaming\nvm); rename the folder to corresponding version (12.20.0 etc.);
4. Use nvm to control the node version.

