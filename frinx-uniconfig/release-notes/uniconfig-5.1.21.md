<!-- Release notes generated using configuration in .github/release.yml at 5.1.21 -->

## What's Changed
### 🐞 Bug Fixes
* Migration for updated logging configuration parameters - 5.1.x-stable
* Fixed unmounting of node that is in connecting state 5.1.x
* Swagger: Fix generation of operational data from Uniconfig schemas
* Swagger: Adjust filter-path for uniconfig models
* Fix deleting existing yang_repository from db
* Fix inserting many yang repositories concurrently
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
* [UNIC-1429] Fix replace operation in GNMI set
* [UNIC-1471] : Fix sync fail after failed installation was stored STABLE
* [UNIC-1474] Improve performance of YANG repository loading process during mounting process (#1785)
* UniConfig Shell - fix prompt callbacks bug
* [UNIC-1492] - Fix rate-limiting
* Fixed loading of gNMI YANG repository during MountNodeTask
* [UNIC-1471] Add schema-cache storing into sync impl
* Prevented sending no description command if there is no change for rpd description
* Gnmi sb netconf cache loader stable
* [UNIC-1494] - add migration for replace-paths
* Fix get fallback schema context in cli shell.
* Fix settings / callbacks cache
* [UNIC-1481] Add With-Default to Java client
* [UNIC-1490] - Fix delete requests not send
* [UNIC-1517] Fix pagination
* [UNIC-1521] Fix reading list nodes in augmentations
* [UNIC-1422] Fixed transaction is closed issue
* [UNIC-1512] Fix ordering of operations on gnmi topology
* Cover more errors in connect node RPC responses
* [UNIC-1526] - increase gnmi executor queue size
* [UNIC-1534] Swagger: fix bad types for numeric attributes (#2018)
* [UNIC-1537] - Fix failing install sync stable
* [UNIC-1547] Fix sync-from-network RPC
* [UNIC-1557] Fix MountPointListenerException
* CVE-2023-51074 stable
* add missing docker dependency
* [UNIC-1573] CLI shell: Fix Connect/Disconnect RPC suggestions
* UniConfig Shell - fixing show mode bugs
* [UNIC-1550] - mount/unmount synchronization (#2061)
* [UNIC-1481] Fix with-defaults query parameter
* Fixed removing mac-address command issue in InterfaceRpdConfigWriter
* [UNIC-1601] Fix loading fallback context.
* Get-template-nodes replace module in var.
* [UNIC-1604] Fix synchronous RPCs - stable
* [UNIC-1620] UC Shell: Fix leaf-list parsing (#2244)
* [UNIC-1629] Fix replace-paths grouping distinct list entries
* [UNIC-1664] Fix confirm-commit race-conditions
* [UNIC-1680] Fix dryrun-commit mount/unmount process
### ✅ New Features
* [UNIC-1394] Client side diff
* [UNIC-1402] UC Shell - default callbacks repository (#1701)
* [UNIC-1075] Uniconfig shell hide / unhide command implementation
* [UNIC-1028] Connect/Disconnect node RPC
* [UNIC-1486] + [UNIC-1509] - Skip yang constraints
* [UNIC-1507] : One uniconfig tx for (un)install-multiple-nodes RPC
* [UNIC-1533] - skip length & range constraints
* Support tailf's comment and label audit params in netconf southbound
* [UNIC-1583] Add timeout property for Uniconfig Shell
* Created CableModemReader and CableModemStateReader for CER devices
* Made local-priority attribute optional in RpdPtpPortConfigWriter
### 💡 Improvements
* checkstyle fixes - 5.1.x-stable
* add ValueCase to gnmi codec - 5.1.x-stable
* Merged mdsal-dom-spi to sal-dom-spi module (5.1.x)
* Enable checkstyle ERROR level by default - 5.1.x-stable
* Removed sal-common-impl module 5.1.x
* Remove stylecheck-logging - 5.1.x-stable
* Swagger: Unit tests
* 5.1.x shell mutable extraction
* Migration to JUnit 5 - part 1 - 5.1.x-stable
* 5.1.x stable verify cancel
* UNIC-1318 - Add option to report yang-patch errors in RFC8040 format - 5.1.x-stable
* JUnit5 migration - part 2 - 5.1.x-stable
* JUnit5 migration - part3 - 5.1.x-stable
* Swagger: Code cleanup - 5.1.x-stable
* 5.1.x stable build speedup
* Shell caching data - 5.1.x-stable
* Shell explicit show in config state (#1484)
* Use native git in git-commit-id plugin - 5.1.x-stable
* Add maven wrapper - 5.1.x-stable
* [UNIC-981] UniconfigShell: Remove explicit show submode from root mod…
* [UNIC-1411] Add mapping to request/response log message
* [UNIC-1394] Add overloaded build methods to client side diff
* [UNIC-1394] exclude gnmi depenendecies from java client
* [UNIC-1401] UniConfig Shell - one line SET / DELETE command (#1621)
* [UNIC-1403] Unified format of shell audit logs (#1658)
* [PANT-83] add logs for pant83 - STABLE
* [UNIC-1408] UniConfig Shell - adjust cached data (#1745)
* [UNIC-1405] UniConfig shell: set nested JSON data (#1752)
* Improved logging (#1798)
* [UNIC-1487] Uniconfig Shell: prompt mode should be the same after config transaction expiration
* [UNIC-1514] Add support CER device version 12.* (#1903)
* Fixed logging in TaskExecutorImpl
* Stable task executor refactoring
* [UNIC-1505] - Parallel syncFromNetwork (stable)
* [UNIC-1102] Improve logging for schema context building
* [UNIC-1499] Swagger: disable generation of POST list entries
* Add postpone reason to task execution error on timeout
* [UNIC-1122] DCE duplicate subscription check.
* Rate limiting for user uniconfig transactions
* [UNIC-1626] Add missing request tags into client.
* Add Docker dependencies for collect_diag_info script
* Added one more command into the update template to remove RPD connection from fiber-node
* Move dependency-check into separate maven profile
### 🖥️ API Changes
* add gnmi-messages logging broker
### 🔨 Dependency Upgrades
* build(deps): dependabot updates 5.1.x-stable
* Bump Spring Boot to 3.1.x - 5.1.x-stable
* Dependabot cherry-picks into 5.1.x-stable
* Suppress FP
* Dependabot cherry-picks - 5.1.x-stable
* Fix Jetty CVEs
* 5.1.x-stable - Maven 3.9.5
* build(deps): bump kotlin.version from 1.9.0 to 1.9.20
* Bump Spring Boot to 3.1.6
* Bump owasp-dependency-check
* Bump logback.classic.version from 1.4.8 to 1.4.14
* Security - bump dependencies and cleanup suppressions
* Security - bump commons-compress to 1.26.0
### 🔧 Other Changes
* 5.1.11-SNAPSHOT
* Gnmi all paths fix 5.1.x
* I did not mean to cherry-pick these
* Unifying, renaming and increasing readability of UC-shell
* Enabling and cleaning up SHELL checkstyles - 5.1.x-stable
* Do not block merge job on main
* Changing use case of prompt stylization
* Release 5.1.11
* 5.1.12-SNAPSHOT
* Workflows: update postgres tests and enable schedule for weekend.
* Shell fix output writing
* Shell fix callbacks set state (#1531)
* Shell fix multiple keys (#1536)
* Add ncurses package to dockerfile
* fix_regex_matching_of_identity
* Release 5.1.12
* 5.1.13-SNAPSHOT
* UniconfigShell: Improving suggestions menu
* [FI-1693] Remove Jenkins-test from merge workflow
* [UNIC-1439] Speed up Device discovery RPC (#1684)
* Revert "[UNIC-1439] Speed up Device discovery RPC (#1684)"
* Release 5.1.13
* 5.1.14-SNAPSHOT
* Release notes workflow for stable branch
* Suppress CVEs
* [UNIC-1475] changing information about expired transaction in root mode
* Release 5.1.14
* New snapshot release 5.1.15
* Do not generate release notes
* [UNIC-1488] Uniconfig Shell: add option to refresh the transaction
* Release version 5.1.15
* Release new snapshot version 5.1.16-SNAPSHOT
* Release workflow stable
* Add distributionSuffix property
* Build distro without units on release (#1955)
* Release 5.1.16.
* Bump to 5.1.17-SNAPSHOT.
* Rename UniConfig distribution name from bin to without TU
* [UNIC-1453] UniConfig Shell - one-liner show/set/delete for callbacks…
* Suppress logback
* [UNIC-1539] Fix some typos
* Maven 3.9.6
* Revert "[UNIC-1547] Fix sync-from-network RPC (#2044)"
* Revert "Revert "[UNIC-1547] Fix sync-from-network RPC (#2044)" (#2050)"
* Suppress sshd vulnerability and old vulnerabilities.
* Release 5.1.17.
* Bump to 5.1.18-SNAPSHOT.
* Add default values to connect-node RPC
* Fixed InterfaceRpdStateReader and CableMacReader for CER devices
* Release 5.1.18.
* Bump to 5.1.19-SNAPSHOT.
* Release 5.1.19.
* Bump to 5.1.20-SNAPSHOT.
* Fix testtool tag
* Release 5.1.20
* Bump to 5.1.21-SNAPSHOT.
* Release 5.1.21
