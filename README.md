# Contents

<!-- TOC -->
* [Contents](#contents)
* [Maven BOM project for snomed projects](#maven-bom-project-for-snomed-projects)
* [Instructions](#instructions)
  * [Option 1) UPDATE VERSION: If there is a fixed version available from the jar/lib supplier.](#option-1-update-version-if-there-is-a-fixed-version-available-from-the-jarlib-supplier)
  * [Option 2) IGNORE CVE: If we wish to ignore the CVE, for whatever reason follow these steps:](#option-2-ignore-cve-if-we-wish-to-ignore-the-cve-for-whatever-reason-follow-these-steps)
  * [Option 3) Exclude the library from the dependency](#option-3-exclude-the-library-from-the-dependency)
* [Useful Maven links:](#useful-maven-links)
* [Useful Maven commands:](#useful-maven-commands)
<!-- TOC -->

# Maven BOM project for snomed projects
- Snomed master Parent BOM project, to centralise and control dependencies more conveniently.
- For OWASP suppressions, see [Ignore CVE section](#option-2-ignore-cve-if-we-wish-to-ignore-the-cve-for-whatever-reason-follow-these-steps), below.
- Remember to update the version in the [pom.xml](pom.xml) with each release.

# Instructions

When a project fails to build due to a CVE there are at least 3 options:

1. An updated version of the library is available and published.
2. An updated version of the library is NOT available.
    - We then have two options:
      - Wait for one to be created, most supported products issue a fix within hours or a few days.
      - Ignore the CVE, see [Ignore CVE section](#option-2-ignore-cve-if-we-wish-to-ignore-the-cve-for-whatever-reason-follow-these-steps), below.
3. Exclude the library from the dependency.

Here are the instructions for each in turn:

## Option 1) UPDATE VERSION: If there is a fixed version available from the jar/lib supplier.

If a project, which uses this BOM fails to build with a CVE error you can follow these instructions.

In summary:
* Create PIP ticket.
* Update version in bom project.
* Update failing project to use the new bom.

In detail:

* Create PIP ticket, include CVE number, and a link to the fix/details.
  - For an example see this old ticket: https://jira.ihtsdotools.org/browse/PIP-257
  - Include link to relevant CVE resolution, for example: https://spring.io/security/cve-2023-20862
* Update the `pom.xml` in this BOM project with the new fix version, here are the XPATHS for the sections you will most likely update:

```text
/project/properties
/project/dependencyManagement/dependencies
```

  - You may need to explicitly specify the module, but mostly it's just a version number change.
  - Update the bom version, here is its XPATH:

```text
/project/version
```

* When the BOM is updated you can test it locally with the affected project:
  - You can push this project locally to test: `mvn install`
  - Update affected projects `pom.xml` to use the new bom version.
  - Run OWASP locally before and after the version change: `mvn dependency-check:check`

* When you are happy push the bom project to git (wait for it to build)
* Then push the affected project, which should now pass.

## Option 2) IGNORE CVE: If we wish to ignore the CVE, for whatever reason follow these steps:

In summary, you will be updating 3 projects:

* Add exception to [snomed-parent-owasp](https://github.com/IHTSDO/snomed-parent-owasp)
* Update version of the OWASP in [snomed-parent-bom](https://github.com/IHTSDO/snomed-parent-bom)
* Update failing project to use the new bom.

Detailed instructions are in the [snomed-parent-owasp](https://github.com/IHTSDO/snomed-parent-owasp) project.

## Option 3) Exclude the library from the dependency
If you do not need the library at all, either do not include it in your `pom.xml`, or if it is from a third party exclude it as follows:

```xml
<dependency>
    <groupId>a.third.party.group</groupId>
    <artifactId>artifact-id</artifactId>
    <exclusions>
        <exclusion>
            <groupId>group.to.exclude</groupId>
            <artifactId>artifact.to.exclude</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

# Useful Maven links:

Again these steps can be tested with various Maven commands, locally.

* Maven Versions Plugin: https://www.mojohaus.org/versions/versions-maven-plugin/index.html
* Maven Dependency Plugin: http://jeremylong.github.io/DependencyCheck/dependency-check-maven/plugin-info.html

# Useful Maven commands:

Note many of the following have corresponding tooling in JetBrains.

* Push project to local repository, for testing locally:

```shell
mvn install
```

* Generates a dependency tree of the project:

```shell
mvn dependency:tree
```

* Display possible updates:

```shell
mvn versions:display-dependency-updates
```

* Searches POM for all -SNAPSHOT versions which have been released and replaces them with the corresponding release version:

```shell
mvn versions:use-releases
```

* Searches POM updates with next version (move every non-SNAPSHOT dependency to its latest release):

```shell
mvn versions:use-next-releases
```

* Searches POM updates with latest version (move every non-SNAPSHOT dependency to its latest release):

```shell
mvn versions:use-latest-versions
```

* Searches POM updates with latest version:

```shell
mvn versions:update-properties
```

* CVE analysis and report:

```shell
mvn dependency-check:check
```
