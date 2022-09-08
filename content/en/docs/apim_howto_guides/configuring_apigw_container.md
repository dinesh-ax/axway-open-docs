{
    "title": "Create an API Gateway with Docker volumes",
    "linkTitle": "Create an API Gateway with Docker volumes",
    "weight": "30",
    "date": "2022-02-01",
    "description": "Configure an API Gateway Docker or Admin Node Manager container to use a persisted volume for configuration."
}

The primary documentation for [API Gateway in containers](/docs/apim_installation/apigw_containers) outlines the creation of images and their deployment as containers. This page describes how during the creation of a container, you can use Docker volumes with a persisted store to update the configuration within the container. The main advantage of using Docker volumes is to allow you to update your configuration without having to rebake the Docker image.

The major steps to create and configure your API Gateway or Admin Node Manager in containers with Docker volumes are:

1. Mount the new configuration.
2. Start the docker container.
3. Verify the container configuration.

This process applies to API Gateway or Admin Node Manager configurations only, such as policy management, system property changes, environment settings, logging configurations, and son on.

## Mount the configuration

Docker volumes facilitate mounting API Gateway or Admin Node Manager configuration files to the Docker container so that they are available to the system at runtime. These configuration files are stored in the host file system, which can be across any hardware or cloud service offering the client is utilizing.

The expected mounted directory (mount point location) structure is as follows:

| Docker volume mount | Description |
| ------------------- | ----------- |
| /merge/fed | For a policy configuration stored as a deployment package (.fed file).|
| /merge/yaml | For YAML based API Gateway policy configuration stored either as a deployment package (.tar.gz file) or as a directory. **Note**: this option is only applicable for API Gateway.|
| /merge/apigateway | For all other API Gateway or Admin Node Manager configurations, for example, jvm.xml, envSettings.props, and so on. |
| /merge/mandatoryFiles | For the verification of mandatory configuration files. |

## Start the container

After you create a persisted store, you must add the Docker volume configuration to the Docker `run` command to start the container with the new runtime configuration. For example:

```bash
# API Gateway
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 \
           -v /home/user/apigw/fed/newFed.fed:/merge/fed \
           -v /home/user/apigw/mandatoryFiles.yaml:/merge/mandatoryFiles \
           -v /home/user/apigw/config:/merge/apigateway \
           -e ACCEPT_GENERAL_CONDITIONS=yes -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-my-group:1.0

# Admin Node Manager
docker run -d -p 8090:8090 --name=anm --network=api-gateway-domain \
           -v /home/user/anm/emt-nm.fed:/merge/fed \
           -v /home/user/anm/mandatoryFiles.yaml:/merge/mandatoryFiles \
           -v /home/user/anm/config:/merge/apigateway \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           -e ACCEPT_GENERAL_CONDITIONS=yes admin-node-manager:0.0.1
```

After the container is running, the files mounted within the Docker volumes are copied to the file system within the running container in its `/merge` directory. For example:

```bash
.
├── merge
│   ├── fed
│   ├──apigateway
│   │  └── groups
│   │      └── emt-group
│   │          └── emt-service
│   │              └── conf
│   │                  ├── envSettings.props
│   │                  └── jvm.xml
│   └── mandatoryFiles
└── opt
    └── Axway
        └── apigateway
```

The original factory configuration in the `/opt/Axway/apigateway` directory is then replaced with the hosted configuration in the `/merge` directory, and the new configuration is reflected in the running instance.

## Verify the configuration

Docker containers use the configuration specified during the image creation process by default. Only files that require to be configured again in the container need to be mounted.

To verify that re-configured files, mounted on Docker volumes, have been successfully found by the Docker container at runtime, you must add the `mandatoryFiles.yaml` configuration file to the `/merge/mandatoryFiles` Docker volume. This file lists the external configuration files which should be mounted to the `/merge` directory in the running container. For example:

```bash
required:
    - /merge/apigateway/groups/emt-group/emt-service/conf/envSettings.props
    - /merge/apigateway/groups/emt-group/emt-service/conf/jvm.xml
    - /merge/apigateway/system/conf/log4j2.yaml2
    - /merge/fed
```

If a file listed in the `mandatoryFiles.yaml` is not available to the container's file system, the container will fail to start and log an appropriate error message, which can be seen by using the `docker logs <container name>` command.

## Sample container configurations

This section provides examples of how to configure an API Gateway or Admin Node Manager Docker container for different configuration types.

### Policy configuration

Policy configuration stored as a deployment package (.fed file) can be added to the Docker container runtime configuration by adding the `fed` file to the `/merge/fed` Docker volume:

```bash
# API Gateway
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 \
           -v /home/user/apigw/fed/newFed.fed:/merge/fed \
           -e ACCEPT_GENERAL_CONDITIONS=yes -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-my-group:1.0

# Admin Node Manager
docker run -d -p 8090:8090 --name=anm --network=api-gateway-domain \
           -v /home/user/anm/emt-nm.fed:/merge/fed \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           -e ACCEPT_GENERAL_CONDITIONS=yes admin-node-manager:0.0.1
```

### YAML Entity Store configuration

YAML based API Gateway policy configuration can be stored either as a deployment package (.tar.gz file) or as a directory. The following examples show how to add them to the Docker container runtime configuration.

{{< alert title="Note" >}}YAML based policy configuration is only applicable for API Gateway.{{< /alert >}}

#### Deployment package

For configuration stored in a deployment package (.tar.gz file), mount the deployment package to the /merge/yaml Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 \
           -v /home/user/apigw/yaml.tar.gz:/merge/yaml \
           -e ACCEPT_GENERAL_CONDITIONS=yes -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-my-group:1.0
```

#### Directory

For configuration stored in a directory, mount the source directory to the /merge/yaml Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 \
           -v /home/user/apigw/yaml:/merge/yaml \
           -e ACCEPT_GENERAL_CONDITIONS=yes -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-my-group:1.0
```

### All other configuration

All other configuration, such as `jvm.xml` and `envSettings.props`, can be added to the Docker container runtime configuration by way of the `/merge/apigateway` Docker volume. You must store these configurations locally, in a directory structure which mirrors `/opt/Axway/apigateway` subfolder structure inside the Docker container. For example:

```bash
config
├── groups
│   └── emt-group
│       └── emt-service
│           └── conf
│               ├── envSettings.props
│               └── jvm.xml
└── system
    └── conf
        └── log4j2.yaml
```

This folder structure is then mounted to the `/merge/apigateway` directory of the container.

After that, you can start API Gateway and Admin Node Manager Docker container as follows:

```
# API Gateway
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 \
           -v /home/user/apigw/config:/merge/apigateway \
           -e ACCEPT_GENERAL_CONDITIONS=yes -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-my-group:1.0

# Admin Node Manager
docker run -d -p 8090:8090 --name=anm --network=api-gateway-domain \
           -v /home/user/anm/config:/merge/apigateway \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           -e ACCEPT_GENERAL_CONDITIONS=yes admin-node-manager:0.0.1
```

## Further Information

For more information on how to build API Gateway or Admin Node Manger Docker images and run in a Docker container, see [Deploy API Gateway in containers](/docs/apim_installation/apigw_containers).
