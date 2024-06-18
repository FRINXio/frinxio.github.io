<!-- Release notes generated using configuration in .github/release.yml at 6.1.0 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1728] Fix reading of sample VNF config from netconf-testtool 
* Add missing dependencies to testtool
* Swagger - add missing namespace when node is augmenting on deeper level
* SwaggerUI - show extensions
* Fix gnmi missing schemaContext handling
* Swagger - fix required statement
* Swagger - fix url parameters
* Docker Compose - add NET_RAW capability
* [UNIC-966] Fix create and update operation in PATCH
* Fix update tag in PATCH and apply-template
* [UNIC-1657] Set found subtree isSelected to true.
* [UNIC-1746] Swagger: fix custom operational path
* [UNIC-1737] Fix synchronization of datastore from DatabaseDOMDataBroker
* Fix sending netconf or gnmi mount body with param schema-cache-directory set to "" (empty string)
* [UNIC-1744] UC Shell: Fix show on choice nodes
### ✅ New Features
* [UNIC-1710] CLI health-check
* Added 13.* version support for ARRIS devices
### 💡 Improvements
* Switch to Exificient
* [UNIC-397] Improve error message for keys in payload.
* [UNIC-1038] Rewrite Dockerfile
* [UNIC-1735] Improve gnmi list deserialization
* Refactor crypto mount parameter to device-crypto in java client side.
* Cleanup dependencies
* Swagger - use oneOf for choice nodes
### 🔨 Dependency Upgrades
* Update wrapper script and mvnw to version 3.3.1
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 1.9.23 to 1.9.24
* Remove unused dependency - objenesis
* Use Embedded Postgres version 16
* Revert "Use Embedded Postgres version 16 (#2419)"
* Docker scout CVE fixes
* Embedded PostgreSQL 16
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.21 to 2.2.22
* build(deps): bump org.codehaus.mojo:build-helper-maven-plugin from 3.5.0 to 3.6.0
* build(deps): bump org.owasp:dependency-check-maven from 9.1.0 to 9.2.0
* build(deps): bump grpc.version from 1.63.0 to 1.64.0
* build(deps): bump org.apache.commons:commons-compress from 1.26.1 to 1.26.2
* build(deps): bump org.codehaus.mojo:exec-maven-plugin from 3.2.0 to 3.3.0
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.16.0 to 10.17.0
* build(deps): bump org.apache.maven.plugins:maven-invoker-plugin from 3.6.1 to 3.7.0
* build(deps): bump commons-cli:commons-cli from 1.7.0 to 1.8.0
* build(deps): bump org.springframework.boot:spring-boot-dependencies from 3.1.11 to 3.1.12
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 1.9.24 to 2.0.0
* build(deps): bump maven.core.version from 3.9.6 to 3.9.7
### 🔧 Other Changes
* Bump to 6.0.4-SNAPSHOT.
* Clean up constants
* Update dependabot.yml
* Revert protocol uppercase back to lowercase.
* Fix typo
* Release 6.1.0.
