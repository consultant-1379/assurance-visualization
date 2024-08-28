# Optionality (Dependency Handling)

[TOC]


This document describes the use of the [optionality.yaml](../charts/__helmChartDockerImageName__/optionality.yaml) file.

The optionality file is used during the Install or Upgrade of the product(s) in which this application resides.

This file is used to declare the external microservice dependencies of an application chart.

Below sections describe the internal workings of microservice dependencies to create an overall optionality file and contains an example of adding a microservice dependency to an optionality file.


There will be a master product-level optionality file (similar to the following located in the EIAE Helmfile Repo [EIAE-optionality.yaml](https://gerrit.ericsson.se/plugins/gitiles/OSS/com.ericsson.oss.eiae/eiae-helmfile/+/HEAD/helmfile/optionality.yaml)) that will contain the mandatory microservices that are used by every application within the product level helmfile.

Each application (SO, PF, DMM etc.) will have their own optionality file that will contain the external microservices for which that specific application depends on.

During the Install or Upgrade, the optionality files from each chart will be combined to create an overall optionality file.

The logic used to create this file is that if a microservice dependency is true within one application optionality file, it will be true for the overall optionality file.

This will ensure that no microservice that an application depends on will be turned off during deployment.


```
optionality:
  eric-cloud-native-base:
    eric-cm-mediator:
      enabled: false
    eric-fh-snmp-alarm-provider:
      enabled: false
    eric-data-document-database-pg:
      enabled: true
    eric-fh-alarm-handler-db-pg:
      enabled: false
  eric-oss-dmm:
    eric-data-message-bus-kf:
      enabled: true
```

The above optionality file shows a dependency on 2 external microservices -
1. eric-data-document-database-pg within the eric-cloud-native-base application.
2. eric-data-message-bus-kf within the eric-oss-dmm application.

NOTE: Setting services to false is not required within application optionality files.
They are only there to clarify what microservices are available in the cloud native base, for example.


Given the example Optionality file above, below is the process of adding a Microservice Dependency.

1. My application has a dependency on the eric-cnom-server which is contained within the eric-oss-common-base chart.
2. In order to declare this dependency the optionality.yaml in my application chart repo has to be updated.
3. The eric-oss-common-base application is not mentioned yet, so we need to add it first.
4. Then under the eric-oss-common-base declaration we can add our specific microservice dependency with the enabled flag set to true as seen below.
5. During the deployment of the product the deployment manager gathers all the optionality files and merges them.
6. The deployment manager then uses this merged optionality file to deploy the product with all the applications and their dependencies enabled.


```
optionality:
  eric-cloud-native-base:
    eric-cm-mediator:
      enabled: false
    eric-fh-snmp-alarm-provider:
      enabled: false
    eric-data-document-database-pg:
      enabled: false
    eric-fh-alarm-handler-db-pg:
      enabled: false
  eric-oss-dmm:
    eric-data-message-bus-kf:
      enabled: true
  eric-oss-common-base:
    eric-cnom-server:
      enabled: true
```
