<!-- Release notes generated using configuration in .github/release.yml at 7.0.1 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1830] Fix constraint check skip for clients
* [UNIC-1828] Log template name and optionally path to config(if present in the origin exception)
* Add TUs for "default-route-distance" container 
* [UNIC-1835] Fix configuration meter filter
* [UNIC-1837] Remove exception throwing when auto templates-upgrader fails
* [UNIC-1839] Remove key from list edit in bulk edit rpc.
### 💡 Improvements
* Move the metrics initialization before the initialization of beans that register metrics to ensure the filter is applied first.
### 🔨 Dependency Upgrades
* build(deps): bump org.apache.maven.plugins:maven-failsafe-plugin from 3.4.0 to 3.5.0
* build(deps): bump org.apache.maven.plugins:maven-surefire-plugin from 3.4.0 to 3.5.0
* build(deps): bump org.apache.maven.plugins:maven-javadoc-plugin from 3.8.0 to 3.10.0
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.14.0 to 3.15.0
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.14.0 to 3.15.0
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.18.0 to 10.18.1
* build(deps): bump com.google.errorprone:error_prone_annotations from 2.30.0 to 2.31.0
* build(deps): bump com.google.errorprone:error_prone_core from 2.30.0 to 2.31.0
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.22 to 2.2.23
* build(deps): bump actions/upload-artifact from 4.3.6 to 4.4.0
* build(deps): bump actions/setup-python from 5.1.1 to 5.2.0
* Add dependency management for version declarations of missing maven p…
* build(deps): bump org.checkerframework:checker-qual from 3.46.0 to 3.47.0
* build(deps): bump org.owasp:dependency-check-maven from 10.0.3 to 10.0.4
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.8.6.2 to 4.8.6.3
* build(deps): bump org.apache.maven.plugins:maven-help-plugin from 3.4.1 to 3.5.0
* build(deps): bump com.github.gantsign.maven:ktlint-maven-plugin from 3.2.0 to 3.3.0
* build(deps): bump com.google.errorprone:error_prone_annotations from 2.31.0 to 2.32.0
* build(deps): bump com.google.protobuf:protobuf-java from 3.25.3 to 3.25.5 in /commons/parents/odlparent
* build(deps): bump com.google.errorprone:error_prone_core from 2.31.0 to 2.32.0
* build(deps): bump org.springframework.boot:spring-boot-dependencies from 3.3.3 to 3.3.4
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.8.6.3 to 4.8.6.4
* build(deps): bump commons-io:commons-io from 2.16.1 to 2.17.0
* build(deps): bump jline.version from 3.26.3 to 3.27.0
### 🔧 Other Changes
* Bump to 7.0.1-SNAPSHOT
* dependency-check - Suppress FP and clean up unused rule
* Document stable branch creation process
* Improve logback for kubernetes clustered deployment
* Add profiling image of Uniconfig
* Release 7.0.1.
