# Tanzu Build Service Documentation

This repository contains content for Tanzu Build Service documentation. We publish the Tanzu Build Service documentation at
[https://docs.pivotal.io/build-service/index.html](https://docs.pivotal.io/build-service/index.html).

## How To Contribute

Please help us improve the accuracy and completeness of the Tanzu Build Service documentation by contributing content, editing,
or expertise.

A common way to contribute is to file a pull request through GitHub.

Every topic in the Tanzu Build Service documentation has a corresponding file in the
[https://github.com/pivotal-cf/docs-build-service](https://github.com/pivotal-cf/docs-build-service) content repository in
GitHub. To locate the source file for a topic, navigate to the topic on the documentation site and click "View
the source for this page in GitHub" at the bottom of the topic.

## Versions and Branching

| **Branch Name** | **Content**               | **Location**                                                                                  |
|-----------------|---------------------------|-----------------------------------------------------------------------------------------------|
| `master`        | DO NOT USE                |                                                                                               |                                                                                     
| `v1.13`         | Tanzu Build Service 1.13  | https://docs-staging.vmware.com/en//Tanzu-Build-Service/1.13/vmware-tanzu-build-service/index.html    |      
| `v1.12`         | Tanzu Build Service 1.12  | https://docs.vmware.com/en/Tanzu-Build-Service/1.12/vmware-tanzu-build-service/index.html     |  
| `v1.11`         | Tanzu Build Service 1.11  | https://docs.vmware.com/en/Tanzu-Build-Service/1.11/vmware-tanzu-build-service/index.html     |        
| `v1.10`         | Tanzu Build Service 1.10  | https://docs.vmware.com/en/Tanzu-Build-Service/1.10/vmware-tanzu-build-service/index.html     |
| `v1.9`          | Tanzu Build Service 1.9   | https://docs.vmware.com/en/Tanzu-Build-Service/1.9/vmware-tanzu-build-service/index.html      |
| `v1.7`          | Tanzu Build Service 1.7   | https://docs.vmware.com/en/Tanzu-Build-Service/1.7/vmware-tanzu-build-service/GUID-index.html |
| `v1.6`          | Tanzu Build Service 1.6   | https://docs.vmware.com/en/Tanzu-Build-Service/1.6/vmware-tanzu-build-service/GUID-index.html |
| `v1.5`          | Tanzu Build Service 1.5   | https://docs.vmware.com/en/Tanzu-Build-Service/1.5/vmware-tanzu-build-service/GUID-index.html |


## Continuous Integration and Continuous Delivery

We use Concourse pipelines to provide continuous integration and continuous delivery. Any change made to this repository
or the [https://github.com/pivotal-cf/docs-build-service](https://github.com/pivotal-cf/docs-build-service) content repository
triggers a "bind" where the disparate parts of the documentation are assembled into a single web app. A successful bind
triggers pushing the app to the staging site,
[https://docs-pcf-staging.cfapps.io/build-service/](http://docs-pcf-staging.cfapps.io/build-service/). After
review, the staging site is manually pushed to the production site,
[https://docs.pivotal.io/platform/build-service/](https://docs.pivotal.io/platform/build-service/).

Concourse Pipeline:

[https://concourse.run.pivotal.io/teams/cf-docs/pipelines/tanzu-build/?group=build-service](https://concourse.run.pivotal.io/teams/cf-docs/pipelines/tanzu-build/?group=build-service)
