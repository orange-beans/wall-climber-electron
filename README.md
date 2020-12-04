# Configure registry for Yarn
##### Yarn config get registry
Set mirror registry:
```sh 
yarn config set registry https://registry.npm.taobao.org
```
Set original registry:
```sh 
yarn config set registry https://registry.yarnpkg.com
```
##### Network connection error
If the following message appears, it means the proxy cannot connect to remote server:
```sh
info There appears to be trouble with your network connection. Retrying...
```
Reset the proxy will solve the problem:
```sh
yarn config delete proxy
npm config rm proxy
npm config rm https-proxy
```
# Install packages for Electron App
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
4. Copy corresponding version of serialport bindings [serialport bindings download link] (https://github.com/serialport/node-serialport/tags). into "app\node_modules\@serialport\bindings\build\Release" folder. 