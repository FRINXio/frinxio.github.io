<!-- Release notes generated using configuration in .github/release.yml at 5.1.22 -->

## What's Changed
### 🐞 Bug Fixes
* Fixed write template in InterfaceRpdConfigWriter
* Added previous mac-address into the command for removing it
* Fixed get-installed-nodes bug that throws error when uninstall-node and get-installed-nodes RPC are executed at the same time in same topology type
### 💡 Improvements
* Prevented sending command for rpd-index, ucam and dcam attributes if there is no change for CER rpd interfaces
### 🔨 Dependency Upgrades
* Security - update dependencies
### 🔧 Other Changes
* upgrade embedded kafka version in application properties
* Release 5.1.22
