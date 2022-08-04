---
title: Release history
linkTitle: Release history
weight: 400
date: 2022-08-04
description: Release history - Changed components.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/UPDATE.md#release-history---changed-components -->

The following table should help you to understand which components have changed with which version. For example, it is not always or very rarely necessary to update Filebeat. If the Elastic version has changed between the some releases, you do not necessarily have to follow it. Of course it is recommended to be on a recent Elastic stack version, because only for this version bugfixes are released by Elastic. Learn here how to update the Elastic stack.

On the other hand, the API builder Docker image, as a central component of the solution, will most likely change with each release.

| Ver   | API-Builder                        | Logstash                           | Memcached                          | Filebeat      | ANM-Config      | Dashboards      | Params          |Elastic-Config      | ELK-Ver.                                 | Notes      |
| :---  | :---:                              | :---:                              | :---:                              | :---:         | :---:           | :---:           | :---:           |:---:               | :---:                                    | :---       |
| 4.6.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.17.3](#update-elastic-stack-version)  |            |
| 4.5.0 | -                                  | [X](#api-builderlogstashmemcached) | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.17.1](#update-elastic-stack-version)  |            |
| 4.4.0 | [X](#api-builderlogstashmemcached) | [X](#api-builderlogstashmemcached) | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.17.1](#update-elastic-stack-version)  |            |
| 4.3.0 | [X](#api-builderlogstashmemcached) | [X](#api-builderlogstashmemcached) | -                                  | -             | -               | [X](#dashboards)| [X](#parameters)|[X](#elastic-config)| [7.17.1](#update-elastic-stack-version)  |            |
| 4.2.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | [X](#dashboards)| [X](#parameters)|-                   | [7.17.0](#update-elastic-stack-version)  |            |
| 4.1.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | [X](#parameters)|[X](#elastic-config)| [7.16.3](#update-elastic-stack-version)  |            |
| 4.0.3 | -                                  | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.16.2](#update-elastic-stack-version)  |            |
| 4.0.2 | -                                  | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.16.1](#update-elastic-stack-version)  | See #154   |
| 4.0.1 | -                                  | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.16.1](#update-elastic-stack-version)  |            |
| 4.0.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | [X](#dashboards)| -               |-                   | [7.15.2](#update-elastic-stack-version)  |            |
| 3.6.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.14.0](#update-elastic-stack-version)  |            |
| 3.5.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.14.0](#update-elastic-stack-version)  |            |
| 3.4.0 | [X](#api-builderlogstashmemcached) | [X](#api-builderlogstashmemcached) | -                                  | -             | -               | [X](#dashboards)| [X](#parameters)|[X](#elastic-config)| [7.14.0](#update-elastic-stack-version)  |            |
| 3.3.2 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.12.1](#update-elastic-stack-version)  |            |
| 3.3.1 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.12.1](#update-elastic-stack-version)  |            |
| 3.3.0 | [X](#api-builderlogstashmemcached) | [X](#api-builderlogstashmemcached) | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.12.1](#update-elastic-stack-version)  |            |
| 3.2.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | -               |[X](#elastic-config)| [7.12.1](#update-elastic-stack-version)  |            |
| 3.1.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | -               |-                   | [7.12.1](#update-elastic-stack-version)  |            |
| 3.0.0 | [X](#api-builderlogstashmemcached) | -                                  | -                                  | -             | -               | -               | [X](#parameters)|-                   | [7.12.1](#update-elastic-stack-version)  |