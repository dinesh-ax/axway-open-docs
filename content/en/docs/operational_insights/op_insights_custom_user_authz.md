---
title: Customize user authorization
linkTitle: Customize user authorization
weight: 35
date: 2022-09-20
description: Customize user authorization in docker compose and helm
---

### Customize user authorization (docker compose)

By default, the organization of the API Manager user determines authorization to the Traffic-Monitor. This means that the user only sees traffic from the organization(s) to which he belongs in API Manager. From a technical point of view, an additional filter clause is added to the Elasticsearch query, which results in a restricted result set. An example: `{ term: { "serviceContext.apiOrg": "Org-A" }}`.

Since version 2.0.0, it is alternatively possible to use an external HTTP service for authorization instead of the API Manager organizations, to restrict the Elasticsearch result based on other criteria.

To customize user authorization, you need to configure an appropriate configuration file as described here:

```bash
# Copy the provided example 
cp ./config/authorization-config-sample.js ./config/myAuthzConfig.js
# Customize your configuration file as needed
vi ./config/myAuthzConfig.js
# Setup your .env file to use your authorization config file
vi .env
AUTHZ_CONFIG=./config/myAuthzConfig.js
# Recreate the API-Builder container
docker stop apibuilder4elastic
docker-compose up -d
```

In this configuration, which also contains corresponding Javascript code, necessary parameters and code is stored, for example to parse the response and to adjust the Elasticsearch query. The example: config/authorization-config-sample.js in the downloaded package contains all required documentation.

Once this configuration is stored, the API Manager Organization based authorization will be replaced.

Note:

* Besides the API Manager Organization authorization only `externalHTTP` is currently supported.
* Only 1 authorization method can be enabled
* It is also possible to disable user authorization completely. To do this, set the parameter: `enableUserAuthorization`: false.
* If you have further use-cases please create an issue describing the use-case/requirements.

### Customize user authorization (helm)

### 1. Create a configMap

Create a ConfigMap that creates your custom configuration file (consult the sample config file config/authorization-config-sample.js): 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: apibuilder4elastic-authz-config
data:
  myAuthzConfig.js: |
    const path = require('path');
    const fs = require('fs');

    /*
        By default, the solution uses user's API Manager organization to determine which 
        API-Requests they are allowed to see in the API Gateway Traffic-Monitor. 
        This behavior can be customized. 
    */

    var authorizationConfig = {
        // For how long should the information cached by the API-Builder process
        cacheTTL: parseInt(process.env.EXT_AUTHZ_CACHE_TTL) ? process.env.EXT_AUTHZ_CACHE_TTL : 300,
        // If you would like to disable user authorization completely, set this flag to false
        enableUserAuthorization: true,
        // Authorize users based on their API-Manager organization (this is the default)
        apimanagerOrganization: {
            enabled: true
        },
    ....
    ..
    .
```

#### 2. Template it

Optionally you may change the generated Yaml file and use it as a Helm template.  

Tip: When using Helm use `.Files.get.` to include your custom configuration file. See here for an example: [templates/elasticApimLogstash/logstash-pipelines.yaml](templates/elasticApimLogstash/logstash-pipelines.yaml)

#### 3. Install or upgrade your setup chart

```
helm upgrade -n apim-elk axway-elk-setup .
Release "axway-elk-setup" has been upgraded. Happy Helming!
NAME: axway-elk-setup
LAST DEPLOYED: Tue May  4 15:06:30 2021
NAMESPACE: apim-elk
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

#### 4. Mount the ConfigFile

Now mount this ConfigMap into the API-Builder container and reference it in the configuration using the `values.yaml`:

```yaml
apibuilder4elastic:
  extraVolumes:
  - name: my-authz-config
    mountPath: /app/config
    subPath: myAuthzConfig.js
    
  extraVolumeMounts:
    - name: my-authz-config
      configMap:
        name: apibuilder4elastic-authz-config
```

Finally, tell API-Builder4Elastic to use your custom configuration:

```yaml
apibuilder4elastic:
  authzConfig: "./config/myAuthzConfig.js"
```
