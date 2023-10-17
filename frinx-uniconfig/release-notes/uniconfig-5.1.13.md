<!-- Release notes generated using configuration in .github/release.yml at 5.1.x-stable -->

## What's Changed
### 🐞 Bug Fixes
* Fix key delimiter in URI
* Swagger: Fix RPCs placed after mountpoint (#1582)
* [UNIC-1410] Fix tx cleanup when request fails
* [UNIC-1413] Fixed updating snapshot in immediate-commit model (#1629)
* [UNIC-1420] Fix cli ssh session reconnect
* [UNIC-1425] Fix crypto bug (#1664)
* [UNIC-1352, UNIC-1254] Fix cluster issues (#1670)
* [UNIC-1340] Fixed releasing of used YANG modules from memory (#1667)
* [UNIC-1429] Fix replace is sent using delete operation
* [UNIC-1432] Swagger: Fix generation of post list endpoints (#1674)
* [UNIC-1430] - fix replace yang-patch for gnmi mountpoint
* [UNIC-1404] UniConfig Shell - fix system augmentation
* Fixed loading of YANG from path in client diff tool
### ✅ New Features
* [UNIC-1394] Client side diff
* [UNIC-1402] UC Shell - default callbacks repository (#1701)
### 💡 Improvements
* [UNIC-981] UniconfigShell: Remove explicit show submode from root mod…
* [UNIC-1411] Add mapping to request/response log message
* [UNIC-1394] Add overloaded build methods to client side diff
* [UNIC-1394] exclude gnmi depenendecies from java client
* [UNIC-1401] UniConfig Shell - one line SET / DELETE command (#1621)
* [UNIC-1403] Unified format of shell audit logs (#1658)
* [PANT-83] add logs for pant83 - STABLE
### 🖥️ API Changes
* add gnmi-messages logging broker
### 🔨 Dependency Upgrades
* Fix Jetty CVEs
* 5.1.x-stable - Maven 3.9.5
### 🔧 Other Changes
* 5.1.13-SNAPSHOT
* UniconfigShell: Improving suggestions menu
* [FI-1693] Remove Jenkins-test from merge workflow
* Release 5.1.13
