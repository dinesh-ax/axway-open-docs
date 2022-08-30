---
title: API Gateway and API Manager 7.7 August 2022 Release Notes
linkTitle: API Gateway and API Manager August 2022
weight: 95
date: 2022-07-11
description: null
---
API Gateway and API Manager updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

In this release, we're adding some user interface (UI) and user experience (UX) improvements, additional support for YAML, externalisation of the configuration for the Admin Node Manager, and the capability to use FIPS with Open SSL 3.0. We keep working to make Amplify API Management a top solution that helps you to control of all your APIs and events across gateways, vendors, and environments!

## Installation

* To **update** your API Gateway, see [Update from API Gateway One Version](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/).
* To **upgrade** from an older version, see [Upgrade from API Gateway 7.5.x or 7.6.x](/docs/apim_installation/apigw_upgrade/upgrade_steps_extcass/).
* For more details on supported platforms for software installation, see [System requirements](/docs/apim_installation/apigtw_install/system_requirements/).
* For a summary of the system requirements for a Docker deployment, see [Set up Docker environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/).

### Update a container deployment

Any custom `.fed` files deployed to a container must be upgraded using [upgradeconfig](/docs/apim_installation/apigw_upgrade/upgrade_analytics#upgradeconfig-options) or [projupgrade](/docs/apim_reference/devopstools_ref#projupgrade-command-options). They must be upgraded the same way, regardless of whether they are API Manager enabled or not. The `.fed` files contain the updates for the API Manager configuration and can be used to build containers.

## New features and enhancements

The following new features and enhancements are available in this update.

### Configure a Node Manager Docker container to use a persisted volume for Node Manager configuration

This feature enhancement allows a Node Manager in container mode to be reconfigured without the need to rebake its Docker image. Configuration updates such as policy changes, JVM system properties, environment properties and so on, can now be mounted on a Docker Volume and made available to a Node Manager container. For more information, see [Create an API Gateway with Docker volumes](/docs/apim_howto_guides/configuring_apigw_container/).

### YAML and Policy Studio

As part of our work to facilitate YAML configuration, with this update YAML is now supported in Policy Studio with specific enhancements made in Environmentalisation.

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update, which may impact your current installation.

### Reintroduction of Federal Information Processing Standards (FIPS)

FIPS capabilities have been reintroduced into API Gateway combined with an upgrade of OpenSSL 3.0.5.

Enabling FIPS restricts cryptographic cipher options pertaining to message encryption and decryption and signature and verification, and it also increases the minimum key length size to those approved by the [Federal Information Processing Standard (140-2)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf).

For more information, see [Restrictions when running in FIPS mode](/docs/apim_administration/apigtw_admin/admin_fips).

Reintroducing FIPS also resulted in changes to the `openssl.cnf` file. The `default_sect` and `legacy_sect` options are now disabled by default in the `INSTALL_DIR/apigateway/conf/openssl.cnf` file.

{{< alert title="Note" color="primary" >}}Upgrading an existing API Gateway will overwrite the `conf/openssl.cnf` file with required FIPS changes. Any custom configuration must be backed up. For more information, see [Upgrade API Gateway](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion){{< /alert >}}

The bundled XML Security Library does not contain the changes required to use the OpenSSL 3.0 FIPS module. As a consequence, XML encryption and signing are not guaranteed to be FIPS compliant.

Hardware Security Module (HSM) usage is not supported in FIPS mode. It is intended that support will be introduced in a future release.

### Policy Studio and Configuration Studio update process

During the Policy Studio and Configuration Studio update process, the `plugins` directory is deleted and then recreated with the new plugins. Therefore, any custom plugins added to this directory is removed.

For more information, see [Install a Policy Studio update](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/#install-a-policy-studio-update) and [Install a Configuration Studio update](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/#install-a-configuration-studio-update).

### HTTP cookie validation

API Gateway validates HTTP cookies as per RFC 6265. A new `com.axway.apigw.cookie.validation.ignore` Java system property has been added to bypass cookie validation introduced in [API Gateway May 2021](/docs/apim_relnotes/20210530_apimgr_relnotes/) release. When the property is set to `true`, API Gateway skips cookie validation.

For more information, see [System property changes](/docs/apim_reference/system_props/).

### HTTP Redirect and Connect to URL filters fail URLs containing non-encoded characters

The **HTTP Redirect** filter now fails URLs containing non-encoded CRLF characters with an `Internal Server Error` instead of passing with a `301 Moved Permanently` response with no location header.

The **Connect to URL filter** now fails URLs containing non-encoded trailing CRLF characters with an `Internal Server Error` instead of passing the trimmed URL without the trailing CRLF characters to the server.

For more information, see [HTTP redirect filter](/docs/apim_policydev/apigw_polref/routing_additional/#http-redirect-filter) and [Connect to URL filter](/docs/apim_policydev/apigw_polref/routing_common/#connect-to-url-filter).

### Organization administrator self-service API publishing

When Self-service API publishing is enabled, Organization administrators can access APIs for the organizations they are a member of or have been granted access to. Previously, when self-service was enabled, API Manager incorrectly allowed access to APIs in other organizations even when no API access was granted.

Customer's scripts or client applications might now fail to get APIs from other organizations if the Organization administrators have not been granted access to these APIs.

For more information, see [API Manager access control, Organization administrator](/docs/api_mgmt_overview/key_concepts/api_mgmt_orgs_roles/index.html#organizationadministrator).

### API Gateway expression language resolver changed

The expression language resolver, which is used to resolve selector statements, was loading multiple expression resolvers into the API Gateway instance, leading to unpredictable behavior. This has been fixed to always use the JUEL API resolver.

### CSP header updated

The default CSP header has been changed to improve security and remove references to unused Axway assets. If you have a custom CSP header defined, see the new defaults.

For API Manager

```bash
script-src 'self' 'unsafe-eval'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; font-src 'self' data: blob:; object-src 'self'; media-src 'self'; frame-src 'self'; frame-ancestors 'none'; upgrade-insecure-requests; manifest-src 'none'; connect-src 'self' https://*:8075 https://*:8065 https://portals-search-api.admin.axway.com; form-action 'self'; prefetch-src 'none'
```

For API Gateway Manager

```bash
script-src 'self' 'unsafe-eval'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; font-src 'self' data: blob:; object-src 'self'; media-src 'self'; frame-src 'self';frame-ancestors 'none'; upgrade-insecure-requests; manifest-src 'none'; connect-src 'self' https://portals-search-api.admin.axway.com; form-action 'self'; prefetch-src 'none'
```

For API Gateway Analytics

```bash
script-src 'self' 'unsafe-eval'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; font-src 'self' data: blob:; object-src 'self'; media-src 'self'; frame-src 'self'; frame-ancestors 'none'; upgrade-insecure-requests; manifest-src 'none'; connect-src 'self'; form-action 'self'; prefetch-src 'none'
```

For more information, see [Define a restrictive Content Security Policy](/docs/apim_installation/apiportal_install/secure_harden_portal/#define-a-restrictive-content-security-policy).

### New Cassandra user script

A new script has been added to help in creating new users in Cassandra. It is located at `apigateway/samples/cassandrauser/scripts/createuser.sh`. For more information, see [Create a new Cassandra database user](/docs/cass_admin/cassandra_config/#create-a-new-cassandra-database-user).

### Disable connection cache for LDAP authentication using Auth Repository

Setting the cache refresh interval of a LDAP authentication via Auth Repository will now disable the cache altogether. For more information, see [General settings in Policy Studio](/docs/apim_reference/general_settings/index.html)

## Deprecated features

No features have been deprecated in this update.

## End of support notices

There are no end of support notices in this update.

## Removed features

<!--To stay current and align our offerings with customer demand and best practices, Axway might discontinue support for some capabilities. As part of this update, the following features have been removed:-->

No features have been removed in this update.

## Fixed issues

This version of API Gateway and API Manager includes:

* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [Release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220830/ipp/100/product/324/product/464/version/3034/version/3035/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

| Internal ID | Case ID                                | Cve Identifier | Description                                                                                                                                                                                                                                                                                     |
| ----------- | -------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|RDAPI-27212|01362214|CVE-2016-9063|**Issue**: API Gateway libexpat version 1.95.7 is vulnerable to CVE-2021-45960.  **Resolution**: libexpat version is now upgraded to 2.4.8, which is not vulnerable to CVE-2021-45960.|
|RDAPI-27290|01365642||**Issue**: When `api.manager.orgadmin.selfservice.enabled`, system property is set to `true`, an organization administrator can see APIs of other organizations. **Resolution**: When self-service is enabled, an organization administrator can only see APIs for organizations they are a member of or have been granted access to.|
|RDAPI-27638|01373636  01372464|CVE-2020-15250|**Issue**: A vulnerability has been found in old dependencies on nimbus-jose-jwt and json-smart. **Resolution**: Upgraded the dependencies to the latest version for json-smart and nimbus-jose-jwt.|
|RDAPI-27710|01379534|CVE-2020-13947|**Issue**: API Gateway is delivered with ActiveMQ 5.16.0, which is vulnerable to CVEs. **Resolution**: ActiveMQ version delivered is now 5.16.5|
|RDAPI-27711|01379534|CVE-2018-14721|**Issue**: Scans of Ehcache version 2.10.6 generates a false positive Security Vulnerability CVE-2018-14721. **Resolution**: Ehcache is now upgraded to version 2.10.9.2 to remove false positive scan results.|

### Other fixed issues

| Internal ID | Case ID                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|RDAPI-19293|01136738|**Issue**: API Catalog Try It only shows the first device in the security profile and is not configurable. **Resolution**: There is now an API Authentication widget on the Try It to allow the user to select which device they would like to use. |
|RDAPI-20742|01167184|**Issue**: Deleting containers of Environmentalized Settings is highly inconsistent when it has more the one child element. **Resolution**: The delete action is now removing all child elements.|
|RDAPI-23233|01233054|**Issue**: The user was not redirected to the login screen after clicking OK on the session timeout dialog. **Resolution**: The user is now redirected to the login screen after clicking OK on the session timeout dialog.|
|RDAPI-25159|01285629|**Issue**: There is no easy way for creating a non-superuser level account for Cassandra access. **Resolution**: You can now create a non-superuser with the required permissions using the script under VORDEL_ROOT/apigateway/samples/cassandrauser/scripts|
|RDAPI-25175|01286047|**Issue**: Inconsistent behavior of API Manager UI Swagger API Import with multiple user sessions. **Resolution**: API Manager UI now updates API Manager settings from the API Manager server periodically based on the configured `polling_interval` in `<INSTALL_DIR>/apigateway/webapps/apiportal/vordel/apiportal/app/app.config` file. Defaults to 15 seconds.|
|RDAPI-26254|01321275|**Issue**: API Key section of the API catalog was not displaying the correct information for API Key Location that was set on the inbound profile. **Resolution**: API Key section of the API catalog is now consistent with the location set on the inbound profile.|
|RDAPI-26696|01337399|**Issue**: An Axway Sentinel link event child-tracked object's name is not the name configured in the Sentinel Cycle Link filter. **Resolution**: Now, an Axway Sentinel link event child tracked object's name is updated to use the Sentinel Cycle Link filter configured name.|
|RDAPI-27000|01385492  01354167  01352724  01363181  01381794  01388240|**Issue**: OAS3 APIs cannot be imported due to the `example` field for parameters in `date-time` format. **Resolution**: OAS3 APIs with the `example` field set for parameters in `date-time` format are now successfully imported. |
|RDAPI-27073|01340727|**Issue**: Open traffic event logs `"status":200` when `recvHeader` is `null`. **Resolution**: The open traffic event now logs the client transaction status code as '0' and text as 'Unexpected' if no response is received from the server. If a response is received from the server, then the received status code and text are logged.|
|RDAPI-27084|01357449|**Issue**: When editing an Application, toggling the quota override when a constraint was in error caused the 'Invalid field' button to remain on screen, preventing the user from navigating back to the main Applications screen. **Resolution**: The validation and event handling for quota overrides have been updated so that the Back button correctly appears when quota overrides are OFF.|
|RDAPI-27394|01362984|**Issue**: A function that was called by managedomain when trying to submit a certificate signed by an External CA was missing an argument. **Resolution**: The missing argument was added, and Certificates can now be submitted successfully in this use-case.|
|RDAPI-27436|01354669|**Issue**: The refresh button on the API Manager > Frontend API screen becomes greyed out when it's clicked in the case where new APIs are created without refreshing the page (such as through an API call), and the button is clicked **Resolution**: This is no longer an issue, and the button no longer becomes greyed out, loading new APIs as appropriate|
|RDAPI-27443|01369583|**Issue**: A Policy Studio/Configuration Studio update does not remove old plugins during  the update. The old plugins are unused by Policy Studio and Configuration Studio, but jars in these unused plugins can still be picked up in security dependency scans. **Resolution**: A Policy Studio/Configuration Studio update now deletes and recreates the plugins folder during the update.|
|RDAPI-27448|01370121|**Issue**: The generated product OAS3 API definitions contain a default value for the LockUserAccount schema that is in an incorrect format ('string' rather than 'object'). **Resolution**: The swagger libraries used to generate the product OAS API definitions have been upgraded so that the LockUserAccount default is of the correct format.|
|RDAPI-27556|01372754|**Issue**: In Policy Studio, when attempting to add a Remote Host Condition to an existing Interface (HTTP/S port) in a YAML configuration, an error was thrown, preventing the condition from being displayed. **Resolution**: In Policy Studio, the treenode decorator for the Ports editor has been updated to correctly retrieve the 'alias' field of the selected Remote Host, thus preventing the error from being thrown and successfully displaying the condition when editing a YAML configuration.|
|RDAPI-27557|01369680|**Issue**: The apigw-backup-tool script fails to restore the tables successfully but shows up the message "Restore Successful" **Resolution**: Fixed the apigw-backup-tool to show up an error message when user data is not populated after restoration.|
|RDAPI-27563|01367621|**Issue**: LDAP authentication via authentication repository does not allow disabling the cache. **Resolution**: LDAP authentication via the authentication repository now allows the cache to be disabled.|
|RDAPI-27599|01375300|**Issue**: Tabs in the WebSocket configuration dialog were missing. **Resolution**: The missing tabs in the WebSocket configuration dialog are now showing again.|
|RDAPI-27630|01377318|**Issue**: API Manager was encoding new lines when storing API tags in KPS. **Resolution**: API Manager now encodes tags without a line break.|
|RDAPI-27663|01376137|**Issue**: Decryption fails when decrypting multiple elements in an encrypted XML with "Removed the EncryptedKey Element used in Decryption" checked. **Resolution**: The encrypted key elements are removed from the DOM after decrypting all encrypted elements.|
|RDAPI-27707|01377113|**Issue**: API Manager API fails to fetch all APIs Swagger 2.0 representation with invalid parameter types in imported WADL APIs. **Resolution**: API Manager now fetches all APIs Swagger 2.0 representation where invalid parameter types are set to string.|
|RDAPI-27803|01367621|**Issue**: Axway doc says LDAP Connection "connectiontimeout" and "cachesize" can be set to zero, but that does not work as expected. **Resolution**: The documentation has been updated to be in sync with the code. The minimum values allowed for "connectiontimeout" and "cachesize are 2000ms and 1 connection, respectively. Anything set less than these values will trigger the default values of 300000 ms (5 mins) and 8 connections.|
|RDAPI-27804|01381544|**Issue**: Query parameter value containing trailing CRLF in HTTP Redirect and Connect to URL filters may be either discarded or set incorrectly. **Resolution**: HTTP Redirect filter now validates the destination URL and fails with an Internal Server Error response for non-encoded CRLF values. Connect to URL filter now fails with an Internal Server Error response for URL with non-encoded trailing CRLF value.|
|RDAPI-27888|01382791  01384839|**Issue**: API Gateway Get Cookie filter no longer allows RFC-invalid cookie values. **Resolution**: A new `com.axway.apigw.cookie.validation.ignore` Java system property has been added allowing to bypass the cookie validation in this filter. Defaults to false.|
|RDAPI-27895|01385143|**Issue**: The 'Upgrade access to newer API' feature in the API Manager UI does not list the available APIs as expected. **Resolution**: The available APIs are now listed when a user tries to upgrade the access of an API.|
|RDAPI-27974|01388854|**Issue**: API docs for Traffic Monitor Swagger was showing incorrect URLs. **Resolution**: API docs now show the updated Swagger with correct URLs.|

## Known issues

The following are known issues for this update.

### EdDSA certificates are not currently supported

TLS and PKI certificates using EdDSA signatures (`ed25519` and `ed448`) for authentication are not currently supported within API Gateway. Certificates cannot be imported into the `config` file, and connections are not be established with external endpoints that use EdDSA certificates.

Related Issue: RDAPI-28033

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
