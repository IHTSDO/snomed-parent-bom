# Maven BOM project for snomed projects
- Snomed master Parent BOM project, to centralise and control dependencies more conveniently
- For OWASP suppressions see the [snomed-parent-owasp](https://github.com/IHTSDO/snomed-parent-owasp) project.
- Remember to update the version in the [pom.xml](pom.xml) with each release.

# Instructions

When a project fails to build due to a CVE there are at least 3 options:

1. An updated version of the library is available and published.
2. An updated version of the library is NOT available.
    - We then have two options:
      - Wait for one to be created, most supported products issue a fix within hours or a few days.
      - Ignore the CVE, See IGNORE CVE section, below.
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

```
/project/properties
/project/dependencyManagement/dependencies
```

  - You may need to explicitly specify the module, but mostly it's just a version number change.
  - Update the bom version, here is its XPATH:

```
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

Here are the detailed instructions:

* Run OWASP Maven goal locally, it should fail: `mvn dependency-check:check`
* However this will have generated a report in the target folder of the project: [target/dependency-check-report.html](target/dependency-check-report.html)
* Open this report and find the CVE you wish to suppress, next to the CVE id there is a supress button.  When clicked you will see an XML snippet, copy this snippet.
* Update the [snomed-parent-owasp](https://github.com/IHTSDO/snomed-parent-owasp) project.
  * Find the `owasp-supressions.xml` and paste the XML snippet at the end after the last suppression.
  * Update the OWASP project version, xpath is `/project/version`
* Update BOM version, to use the new OWASP version, xpath in the bom project is `/project/properties/snomed-parent-owasp.version`
* Update affected project, to use the new BOM version.

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

`mvn install`

* Generates a dependency tree of the project:

`mvn dependency:tree`

* Display possible updates:

`mvn versions:display-dependency-updates`

* Searches POM for all -SNAPSHOT versions which have been released and replaces them with the corresponding release version:

`mvn versions:use-releases`

* Searches POM updates with next version (move every non-SNAPSHOT dependency to its latest release):

`mvn versions:use-next-releases`

* Searches POM updates with latest version (move every non-SNAPSHOT dependency to its latest release):

`mvn versions:use-latest-versions`

* Searches POM updates with latest version:

`mvn versions:update-properties`

* CVE analysis and report:

`mvn dependency-check:check`

