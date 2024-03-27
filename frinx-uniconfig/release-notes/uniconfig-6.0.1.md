<!-- Release notes generated using configuration in .github/release.yml at 6.0.1 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1631] Fix EscapedKeysUriFormatter
* [UNIC-1612] Fix serialization path in in json filter serializer
* Fix mount-node task rate-limiting
* [UNIC-1586] Fix jsonb filter data util.
* [UNIC-1620] UC Shell: Fix leaf-list parsing
* [UNIC-1667] Add gnmi models to OpenAPI docs
* [UNIC-1629] Fix replace-paths grouping distinct list entries
* [UNIC-1603] Gnmi - Fix getting closest parent for augmentation node.
* [VHD-377] Rename mountpoint paramater crypto to device-crypto
* [VHD-380] Add missing RequestErrorTag.INVALID_TRANSACTION to client side.
* Fix handling of fingerprint read in checked-commit
* [VHD-377] Change qname 'crypto' to 'device-crypto'.
* [UNIC-1611] Add missing return statement.
* [UNIC-1679] Fix IOS XR parsing
* [UNIC-1663] Revert fix lazy apply template
* [UNIC-1682] Fix augmentation nodes overwriting each other
* Fix casting of UDE
### ✅ New Features
* Made local-priority attribute optional in RpdPtpPortConfigWriter
* [UNIC-1684] - Add read and update properties client implementation
### 💡 Improvements
* [UNIC-1626] Add missing request tags into client.
* [UNIC-1614] Adjust Connect/Disconnect RPC for better user experience
* [UNIC-1616] - add node-id into error-info by default
* Add logs to warn user when MIB repository metadata is missing
* Add Docker dependencies for collect_diag_info script
* [UNIC-1643] Add gnmi subscription parameters
* Added one more command into the update template to remove RPD connection from fiber-node
* Update gitignore
* [UNIC-1606] Skip yang-patch exist check operation
* [VHD-363] Improving response if there is no content with using count.
* Add EdDSA dependency
* Separate vulnerability scanning into a workflow
* Add slack notification to vulnscan workflow
* Fix notification URL
* Fix vulnscan ids
### 🔨 Dependency Upgrades
* build(deps): bump actions/download-artifact from 4.1.2 to 4.1.4
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.13.0 to 10.14.0
* build(deps): bump com.github.gantsign.maven:ktlint-maven-plugin from 3.0.0 to 3.1.1
* build(deps): bump org.apache.maven.plugins:maven-assembly-plugin from 3.6.0 to 3.7.0
* build(deps): bump io.github.git-commit-id:git-commit-id-maven-plugin from 7.0.0 to 8.0.1
* build(deps): bump org.jetbrains.dokka:dokka-maven-plugin from 1.9.10 to 1.9.20
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 1.9.22 to 1.9.23
* build(deps): bump grpc.version from 1.61.1 to 1.62.2
* build(deps): bump org.json:json from 20240205 to 20240303
* build(deps): bump org.apache.commons:commons-compress from 1.26.0 to 1.26.1
* build(deps): bump org.apache.maven.plugins:maven-remote-resources-plugin from 3.1.0 to 3.2.0
* build(deps): bump args4j:args4j from 2.33 to 2.37
* build(deps): bump com.github.gantsign.maven:ktlint-maven-plugin from 3.1.1 to 3.2.0
* build(deps): bump org.owasp:dependency-check-maven from 9.0.9 to 9.0.10
* build(deps): bump com.google.errorprone:error_prone_core from 2.25.0 to 2.26.1
* build(deps): bump org.springdoc:springdoc-openapi-starter-webmvc-ui from 2.3.0 to 2.4.0
* build(deps): bump com.google.guava:guava from 33.0.0-jre to 33.1.0-jre
* build(deps): bump docker/login-action from 3.0.0 to 3.1.0
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.14.0 to 10.14.2
* build(deps): bump org.apache.maven.plugins:maven-compiler-plugin from 3.12.1 to 3.13.0
* build(deps): bump org.apache.maven.plugins:maven-assembly-plugin from 3.7.0 to 3.7.1
* build(deps): bump io.github.git-commit-id:git-commit-id-maven-plugin from 8.0.1 to 8.0.2
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.20 to 2.2.21
* build(deps): bump org.springframework.boot:spring-boot-dependencies from 3.1.9 to 3.1.10
### 🔧 Other Changes
* Bump to 6.0.1-SNAPSHOT.
* Fix collect_diag_info.sh
* Fix logging class
* Update vulnscan.yml
* Release 6.0.1
