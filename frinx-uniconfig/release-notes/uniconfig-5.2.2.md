<!-- Release notes generated using configuration in .github/release.yml at 5.2.2 -->

## What's Changed
### 🐞 Bug Fixes
* Caching request body copier
* Use a Set<path> instead of a List<Path> in MibRepository
* [UNIC-1405] UniConfig shell: set nested JSON data
* [UNIC-1450] Spotbugs fixes - Bad practice
* UniConfig Shell - fix prompt callbacks bug
* [UNIC-1429] Fix replace operation in GNMI set (MAIN)
* [UNIC-1471] : Fix sync fails after failed installation was stored in DB
* [UNIC-1474] Improve performance of YANG repository loading process during mounting process
* Fix exception in loading yang schemas
* Fix update property value to null bug
* [UNIC-1475]: generalizing information about expired transaction
* [UNIC-1471] Add schema-cache storing into sync impl
* Fix get fallback schema context in cli shell.
* [UNIC-1494] - add migration for replace-paths
* Removed the forgotten callbacks-models dependencies
* Fix bad migration embedded kafka properties from old UC version to new
* Set forgotten crypto properties in creation crypto config.
* Add fix for reading duplicate properties
### ✅ New Features
* [UNIC-1075] Uniconfig shell hide / unhide command implementation
* [UNIC-1028] Connect/Disconnect node RPC
### 💡 Improvements
* [UNIC-1408] UniConfig Shell - adjust cached data
* [UNIC-1374] Fixed sending install-node RPC request without mandatory fields
* [UNIC-1445] Refactor yang-repo to JOOQ style
* Refactoring Properties
* Add spotbugs-maven-plugin configuration
* Optimize DB read-only transaction
* Improved logging
* Add Google ErrorProne plugin
* [UNIC-1487] return to the same mode when transaction expires
### 🖥️ API Changes
* [UNIC-1290] Refactor data-change-events RPCs
### 🔨 Dependency Upgrades
* build(deps): bump com.github.spotbugs:spotbugs-annotations from 4.7.3 to 4.8.0
* build(deps): bump com.google.guava:guava from 32.1.2-jre to 32.1.3-jre
* build(deps): bump org.codehaus.woodstox:stax2-api from 4.2.1 to 4.2.2
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.16 to 2.2.17
* build(deps): bump com.fasterxml.jackson.core:jackson-databind from 2.15.2 to 2.15.3
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.9.0 to 3.10.1
* build(deps): bump org.jetbrains.dokka:dokka-maven-plugin from 1.9.0 to 1.9.10
* build(deps): bump io.github.git-commit-id:git-commit-id-maven-plugin from 6.0.0 to 7.0.0
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.9.0 to 3.10.1
* build(deps): bump grpc.version from 1.58.0 to 1.59.0
* build(deps): bump spring.boot.version from 3.1.4 to 3.1.5
* build(deps): bump sshd.version from 2.10.0 to 2.11.0
* build(deps): bump org.owasp:dependency-check-maven from 8.4.0 to 8.4.2
### 🔧 Other Changes
* Rename distro/uniconfig-modules/uniconfig to main
* Prepared sample docker compose file
* Move main.jar into the root
* Set kafka enabled to false
* Properties overhaul
* Suppress CVEs
* Use NetconfCacheLoader in gnmi-sb instead of custom yang parsing
* Fix binding empty properties
* Rewrite client to new properties RPC
* Fix dependency management for starting UniConfig
* Release 5.2.2
