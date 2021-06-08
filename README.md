# Tanzu Build Service Documentation

This repository contains content for Tanzu Build Service documentation. We publish the Tanzu Build Service documentation at
[https://docs.pivotal.io/build-service/index.html](https://docs.pivotal.io/build-service/index.html).
The repository for the book buil

## How To Contribute

Please help us improve the accuracy and completeness of the Tanzu Build Service documentation by contributing content, editing,
or expertise.

A common way to contribute is to file a pull request through GitHub.

Every topic in the Tanzu Build Service documentation has a corresponding file in the
[https://github.com/pivotal-cf/docs-build-service](https://github.com/pivotal-cf/docs-build-service) content repository in
GitHub. To locate the source file for a topic, navigate to the topic on the documentation site and click "View
the source for this page in GitHub" at the bottom of the topic.

## Versions and Branching

| **Branch Name** | **Content** | **Location** |
|-----------------|-------------|--------------|
| `master` | DO NOT USE | |
| `v1.2`   | Tanzu Build Service 1.2    | https://docs.pivotal.io/build-service/1-2/ |
| `v1.1`   | Tanzu Build Service 1.1    | https://docs.pivotal.io/build-service/1-1/ |
| `v1.0`   | Tanzu Build Service 1.0    | https://docs.pivotal.io/build-service/1-0/ |

## Where is the book repository?

The  book repo associated with this content repo is [**docs-book-buildservice**](https://github.com/pivotal-cf/docs-book-buildservice).

## Continuous Integration and Continuous Delivery

We use Concourse pipelines to provide continuous integration and continuous delivery. Any change made to this repository
or the [https://github.com/pivotal-cf/docs-build-service](https://github.com/pivotal-cf/docs-build-service) content repository
triggers a "bind" where the disparate parts of the documentation are assembled into a single web app. A successful bind
triggers pushing the app to the staging site,
[https://docs-pcf-staging.cfapps.io/build-service/](http://docs-pcf-staging.cfapps.io/build-service/). After
review, the staging site is manually pushed to the production site,
[https://docs.pivotal.io/platform/build-service/](https://docs.pivotal.io/platform/build-service/).

Concourse Pipeline:

[https://runway-ci.eng.vmware.com/teams/mapbu-docs/pipelines/tanzu-build](https://runway-ci.eng.vmware.com/teams/mapbu-docs/pipelines/tanzu-build)
