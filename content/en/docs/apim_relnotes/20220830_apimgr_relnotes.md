---
title: API Gateway and API Manager 7.7 August 2022 Release Notes
linkTitle: API Gateway and API Manager August 2022
weight: 95
date: 2022-07-11
description: null
---
API Gateway and API Manager updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

This latest release continues to simplify and evolve API management...

placeholder

## Installation

* To **update** your API Gateway, see [Update from API Gateway One Version](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/).
* To **upgrade** from an older version, see [Upgrade from API Gateway 7.5.x or 7.6.x](/docs/apim_installation/apigw_upgrade/upgrade_steps_extcass/).
* For more details on supported platforms for software installation, see [System requirements](/docs/apim_installation/apigtw_install/system_requirements/).
* For a summary of the system requirements for a Docker deployment, see [Set up Docker environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/).

### Update a container deployment

Any custom `.fed` files deployed to a container must be upgraded using [upgradeconfig](/docs/apim_installation/apigw_upgrade/upgrade_analytics#upgradeconfig-options) or [projupgrade](/docs/apim_reference/devopstools_ref#projupgrade-command-options). They must be upgraded the same way, regardless of whether they are API Manager enabled or not. The `.fed` files contain the updates for the API Manager configuration and can be used to build containers.

## New features and enhancements

The following new features and enhancements are available in this update.

### placeholder 1

placeholder

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update, which may impact your current installation.

### Policy Studio and Configuration Studio update process

During the Policy Studio and Configuration Studio update process, the `plugins` directory is deleted and then recreated with the new plugins.

For more information, see [Install a Policy Studio update](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/#install-a-policy-studio-update) and [Install a Configuration Studio update](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/#install-a-configuration-studio-update).

## Deprecated features

As part of our software development life cycle, we constantly review our API Management offering. As part of this update, the following capabilities have been deprecated

### placeholder 3

placeholder 3

## End of support notices

There are no end of support notices in this update.

## Removed features

<!--To stay current and align our offerings with customer demand and best practices, Axway might discontinue support for some capabilities. As part of this update, the following features have been removed:-->

No features have been removed in this update.

## Fixed issues

This version of API Gateway and API Manager includes:

* Fixes from all 7.5.3, 7.6.2, and 7.7 service packs released prior to this version. For details of all the service pack fixes included, see the corresponding *SP Readme* attached to each service pack on [Axway Support](https://support.axway.com).
* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [Release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220530/ipp/100/product/324/product/464/version/3034/version/3035/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

table

### Other fixed issues

table

## Known issues

The following are known issues for this update.

### API Analytics PDF reports do not display chart contents

In API Analytics, the PDF reports no longer correctly display the contents of the charts. This issue has arisen due to a security upgrade of the `Highcharts.js` charting library. We are working on the fix of this functionality, to be released in a future update of API Gateway.

Related Issue: RDAPI-27301

### Scripting filter whiteboard attributes not preloaded for Jython scripts

The Scripting filter now uses a Jython 2.7 scripting environment (previously, Jython 2.5) to execute Jython scripts. As a result of this version change, the whiteboard attributes, such as `http.request.uri` and `http.request.verb`, are no longer preloaded for use by Jython scripts. However, you can run a Jython script to load these attributes before they are accessed as follows:

```
from com.vordel.trace import Trace

def invoke(msg):
    msg.forceGenerateAttributes()
    Trace.info("This trace statement was generated in script filter!  [" + str(msg.get("http.request.verb")) + "] [" + str(msg.get("http.request.uri")) + "]")
    return True
```

Related Issue: RDAPI-21363

### When an API Gateway instance is started, Xerces SAXParserImpl writes warnings to the error console

At API Gateway instance startup, the following warnings are logged to the error console, as opposed to the trace log:

```
Warning: org.apache.xerces.jaxp.SAXParserImpl$JAXPSAXParser: Property 'http://javax.xml.XMLConstants/property/accessExternalDTD' is not recognized.
Warning: org.apache.xerces.jaxp.SAXParserImpl$JAXPSAXParser: Property 'http://www.oracle.com/xml/jaxp/properties/entityExpansionLimit' is not recognized.
```

These new properties were added in JAXP 1.5 specification, which is supported by the embedded implementation in the JRE but not supported yet in Xerces-J Apache implementation. These are harmless warning messages, which are written to the error console instead of throwing an exception if a property is not supported by the Apache Xerces-J implementation.

Related Issue: RDAPI-22218

### API Gateway web service WSDL schema validation failure

If a web service is defined using multiple WSDLs, an error of 'Cannot find the declaration of element' might occur during the schema validation of a SOAP message. This might happen because of a duplication of the WSDLs types schema `targetNamespace`. To avoid this failure, you must change the types schema `targetNamespace` to be unique across the WSDLs.

Related Issue: RDAPI-26621

### API Catalog Swagger 2.0 export issue when multiple API Manager traffic ports configured

When exporting Swagger 2.0 from API Manager Catalog with multiple traffic ports defined, if you try to subsequently reimport the generated document into another API Manager, the back-end URLs (HTTP and HTTPS) are correctly generated but the port will be incorrect for one of the URLs.

The issue is caused by an inherent flaw in Swagger 2.0 as it only permits one host to be specified. At export time, API Manager takes the first SSL-based basePath, if one exists, from the list of available basePaths (host:port) and applies that to the `$.host` field in the resultant Swagger 2.0 definition.

For example, if an HTTPS traffic port of `8065` and an HTTP traffic port of `8066` are configured, and the host IP address is `127.0.0.1`, then the generated Swagger 2.0 definition will look like this:

```json
{
  "swagger" : "2.0",
  "info" : {
    "description" : "",
    "version" : "1.0.3",
    "title" : "Test API"
  },
  "host" : "127.0.0.1:8065",
  "basePath" : "/api",
  "schemes" : [ "https", "http" ],
  "paths" : {...}
}
```

Note that the `$.host` field specifies a port of 8065. Importing this definition back into API Manager will result in the following back-end URLs, with the HTTP port being incorrect.

* `https://127.0.0.1:8065/api`
* `http://127.0.0.1:8065/api`

The recommendation is to use OpenAPI, which has the ability to specify multiple back-end hosts. For more information, see [OpenAPI Specification, Server Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.2.md#serverObject).

Related Issue: RDAPI-23379

### When multiple API Manager traffic ports are configured, specifying a virtual host that contains a host and port can cause conflicting base path URLs to be displayed in API Catalog

In API Manager, if a virtual host (global default, organization level, or for a published API) is set to `myhost` and there are multiple traffic ports (mix of HTTP and HTTPS) configured, API Manager correctly displays `https://myhost` and `http://myhost` as base path URLs in the API Catalog.

An issue only arises when a port is specified as part of the virtual host. API Manager blindly takes the specified virtual host and appends it to the supported schemes for the configured traffic ports. So if a virtual host of `myhost:9999` is set, then conflicting base paths of `https://myhost:9999` and `http://myhost:9999` are displayed in the API Catalog.

Related Issue: RDAPI-23379

## Documentation

To find all available documentation for this product version:

1. Go to [Manuals on the Axway Documentation portal](https://docs.axway.com/bundle).
2. In the left pane **Filters** list, select your product or product version.

Customers with active support contracts must log in to access restricted content.

For information on the different operating systems, databases, browsers, and thick client platforms supported by each Axway product, see [Supported Platforms](https://docs.axway.com/bundle/Axway_Products_SupportedPlatforms_allOS_en).

## Support services

The Axway Global Support team provides worldwide 24 x 7 support for customers with active support agreements.

Email [support@axway.com](mailto:support@axway.com) or visit [Axway Support](https://support.axway.com/).

See [Get help with API Gateway](/docs/apim_administration/apigtw_admin/trblshoot_get_help/) for the information that you should be prepared to provide when you contact Axway Support.
