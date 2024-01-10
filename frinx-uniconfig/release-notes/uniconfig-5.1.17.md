<!-- Release notes generated using configuration in .github/release.yml at 5.1.17 -->

## What's Changed
### 🐞 Bug Fixes
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
### ✅ New Features
* [UNIC-1486] + [UNIC-1509] - Skip yang constraints
* [UNIC-1507] : One uniconfig tx for (un)install-multiple-nodes RPC
* [UNIC-1533] - skip length & range constraints
* Support tailf's comment and label audit params in netconf southbound
### 💡 Improvements
* [UNIC-1514] Add support CER device version 12.* (#1903)
* Fixed logging in TaskExecutorImpl
* Stable task executor refactoring
* [UNIC-1505] - Parallel syncFromNetwork (stable)
* [UNIC-1102] Improve logging for schema context building
### 🔨 Dependency Upgrades
* Bump Spring Boot to 3.1.6
* Bump owasp-dependency-check
* Bump logback.classic.version from 1.4.8 to 1.4.14
### 🔧 Other Changes
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
