---
name: java-webapp-module-development
description: Use when a scientific data infrastructure (e.g., spectral or metadata
  databases) must be implemented or extended as a Java Web application with REST endpoints,
  a Web interface, and a Maven/Tomcat build-deploy chain.
when_to_use_negative:
- The task is standalone spectral processing or NMR data manipulation — use dedicated
  Python packages (e.g., NMRPro) rather than building a Java webapp module
- PeakForest is designed for metabolite annotation report data rather than storage
  of raw spectral data, so raw-data storage backends are out of scope
- The team lacks computing/bioinformatics skills, since technical installation requires
  such personnel
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3314
tools:
- name: PeakForest
  role: Reference open-source Java Web application and API toolbox whose webapp module
    (peakforest-webapp) demonstrates the build/test/deploy pattern
  repo: https://github.com/peakforest/peakforest-webapp
- name: Maven 4+
  role: 'Build tool: `mvn clean install` produces the WAR artifact; `mvn test` runs
    the unit tests'
- name: Tomcat 7
  role: Servlet server hosting the deployed webapp WAR and the REST documentation
- name: Java 8
  role: Runtime JVM version (1.8+) required to run the application
- name: MySQL 5+
  role: Backend relational database used by the Hibernate-configured webapp
- name: NMRPro
  role: Python NMR spectra viewer isolated in a Docker container (npaulhe/nmrpro)
    and integrated as a JS tool in the webapp
  repo: https://github.com/ahmohamed/nmrpro
- name: Open Babel
  role: Binary chemistry toolbox (version 2.3.2 with java bundle) required as a dependency
    of the webapp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:7205b0115b02b57c839b771239b816892b8a6dd980f8e4ff5c91a1a0603ca682
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java
  - Java 8
  - Maven 4+
  - MySQL 5+
  - NMRPro
  - Open Babel
  - PeakForest
  - Tomcat 7
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# java-webapp-module-development

## Summary

Develop, build, test, and deploy a Java-based scientific web application module following the PeakForest webapp pattern: a Maven-built Java 8 Web application packaged as a WAR for Tomcat deployment with REST Web services, configuration-file management, and unit-tested code.

## When to use

Use when a scientific data infrastructure (e.g., spectral or metadata databases) must be implemented or extended as a Java Web application with REST endpoints, a Web interface, and a Maven/Tomcat build-deploy chain.

## When NOT to use

- The task is standalone spectral processing or NMR data manipulation — use dedicated Python packages (e.g., NMRPro) rather than building a Java webapp module
- PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data, so raw-data storage backends are out of scope
- The team lacks computing/bioinformatics skills, since technical installation requires such personnel

## Inputs

- Java webapp source project cloned from Git (e.g., peakforest-webapp)
- Configuration files: src/main/resources/conf.properties, info.properties, hibernate.cfg.xml, server/metExploreData.
- Curated compound and spectral metadata records (JSON/CSV) to serve through the module

## Outputs

- peakforest-webapp-X.war — Java binary ready for Tomcat deployment
- Deployed Java Web application with REST Web services (compound library manager, spectra library manager, spectrum viewers, curation interface, admin interface)
- mvn test unit-test execution results

## How to apply

Set up the Java environment (JVM 1.8+), Maven 4+, and MySQL 5+ as required by the PeakForest webapp, plus Open Babel 2.3.2 with the java bundle when chemical structure conversion is needed. Clone the project Git repository and build with `mvn clean install`, which produces a deployable `peakforest-webapp-X.war` artifact for a Tomcat 7 server. Run `mvn test` to execute the unit-test suite before deployment, and configure the application via the resource files `conf.properties`, `info.properties`, `hibernate.cfg.xml`, and `metExploreData.`. Design the module as an API toolbox (as PeakForest does with its REST Web services and external-banks API) so that curated data — e.g., MS/NMR spectral metadata — can be imported, curated, and exploited via data export or connected Web services. Deploy the WAR to Tomcat and expose services such as compound/spectra library management, spectrum viewers, curation, and admin interfaces; isolate heavyweight helper components (like the Python NMRPro viewer) in Docker containers to avoid socket conflicts.

## Related tools

- **PeakForest** (Reference open-source Java Web application and API toolbox whose webapp module (peakforest-webapp) demonstrates the build/test/deploy pattern) — https://github.com/peakforest/peakforest-webapp
- **Maven 4+** (Build tool: `mvn clean install` produces the WAR artifact; `mvn test` runs the unit tests)
- **Tomcat 7** (Servlet server hosting the deployed webapp WAR and the REST documentation)
- **Java 8** (Runtime JVM version (1.8+) required to run the application)
- **MySQL 5+** (Backend relational database used by the Hibernate-configured webapp)
- **NMRPro** (Python NMR spectra viewer isolated in a Docker container (npaulhe/nmrpro) and integrated as a JS tool in the webapp) — https://github.com/ahmohamed/nmrpro
- **Open Babel** (Binary chemistry toolbox (version 2.3.2 with java bundle) required as a dependency of the webapp)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn clean install && mvn test
```

## Evaluation signals

- `mvn clean install` completes successfully and emits a `peakforest-webapp-X.war` file whose version number X matches the project version
- `mvn test` launches all unit tests without failures (test classes/packages selectable in STS or Eclipse)
- The WAR deploys cleanly on a Tomcat 7 server and the Web application plus REST documentation are served
- Configuration files (conf.properties, info.properties, hibernate.cfg.xml, metExploreData.) are present and correctly populated for the target environment
- Exported/served records preserve compound metadata: the number of exported records equals the number of spectra in the input collection

## Limitations

- Technical installation in a laboratory information system requires personnel with computing or bioinformatics skills
- A minimum number of compounds and spectra must be imported by users before local databases are fully functional; initial database construction takes time and deployment is iterative
- Developers must open the MySQL socket locally for tests; socket conflicts are avoided by wrapping the NMRPro viewer in a Docker container
- Open Babel and database configuration require consulting the separate developer documentation

## Evidence

- [methods] PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1).: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [readme] Build command `mvn clean install` produce: - `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy; `X` matching the current version number): "Build command `mvn clean install` produce:
   - `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy; `X` matching the current version number)"
- [readme] This project uses and requires: - java JVM 1.8+ - maven 4+ - MySQL 5+: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [readme] In order to avoid install issues and socket conflict, it is isolated in a Docker container.: "In order to avoid install issues and socket conflict, it is isolated in a Docker container."
