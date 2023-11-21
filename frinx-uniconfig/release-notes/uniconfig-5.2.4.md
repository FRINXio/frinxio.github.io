<!-- Release notes generated using configuration in .github/release.yml at 5.2.4 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1517] Fix pagination
* Fix getting fallback schema token.
### ✅ New Features
* [UNIC-1486] [UNIC-1509] Skip yang constraints 
### 💡 Improvements
* [UNIC-1514] Add support CER device version 12.*
* [DEP-616] UDP loadbalancer with ip transparency for SNMP notifications
* [UNIC-1495] Refactor mountpoint to JOOQ style
* Add new Release workflow
### 🖥️ API Changes
* [UNIC-1288] Refactor logging and restconf-logging RPCs
### 🔨 Dependency Upgrades
* build(deps): bump com.github.spotbugs:spotbugs-annotations from 4.8.0 to 4.8.1
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.7.3.6 to 4.8.1.0
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.18 to 2.2.19
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.10.1 to 3.10.2
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.10.1 to 3.10.2
* build(deps): bump io.zonky.test.postgres:embedded-postgres-binaries-linux-amd64 from 13.12.0 to 13.13.0
### 🔧 Other Changes
* Release new snapshot version 5.2.4-SNAPSHOT
* Workflows: enable immutable and mutable cluster tests.
* [UNIC-1407] Adjust LOGs in Uniconfig shell
* Workflows: add new test to embeded workflow - skip_yang_constraints.
* Workflows: fix typo for embeded tests.
* [UNIC-1438] Refactoring sal-fallback-provider / fallback-schema-service
* Fixed logging in TaskExecutorImpl
* Add name to release workflow
* Allow double quotes in release notes body
* Use PAT in release workflow
* Workflows: fix typo in embededDB_cross_versions workflow.
* Fix PAT usage
* Dispatch on wf completion instead
* Workflows: add cache for embeded test and remove all docker images
* Merge publish release notes into release wf as a separate job
* Fix release workflow
* Update release.yml
