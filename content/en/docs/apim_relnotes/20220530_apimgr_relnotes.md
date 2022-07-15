---
title: API Gateway and API Manager 7.7 May 2022 Release Notes
linkTitle: API Gateway and API Manager May 2022
weight: 95
date: 2022-03-22
description: null
---
API Gateway and API Manager updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

This latest release continues to simplify and evolve API management. Among the updates to help reduce the burden of API management is the API manager user interface performance improvement when dealing with a large number of APIs, a newly enhanced set of redaction rules, and an upgrade to Open SSL 3.0. Also included in this update, users will experience better control of timeout for Connect Filters, enhanced externalization of configuration for containerized deployments, and better Policy Studio performance when using on Windows operating systems.

## Installation

* To **update** your API Gateway, see [Update from API Gateway One Version](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/).
* To **upgrade** from an older version, see [Upgrade from API Gateway 7.5.x or 7.6.x](/docs/apim_installation/apigw_upgrade/upgrade_steps_extcass/).
* For more details on supported platforms for software installation, see [System requirements](/docs/apim_installation/apigtw_install/system_requirements/).
* For a summary of the system requirements for a Docker deployment, see [Set up Docker environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/).

### Update a container deployment

Any custom `.fed` files deployed to a container must be upgraded using [upgradeconfig](/docs/apim_installation/apigw_upgrade/upgrade_analytics#upgradeconfig-options) or [projupgrade](/docs/apim_reference/devopstools_ref#projupgrade-command-options). They must be upgraded the same way, regardless of whether they are API Manager enabled or not. The `.fed` files contain the updates for the API Manager configuration and can be used to build containers.

## New features and enhancements

The following new features and enhancements are available in this update.

### Manage timeouts on Connection and ConnectToUrl filters

**Timeout** settings can now be set on the **Connection** and **ConnectToUrl** filters. This change allows for specific calls to timeout at different times based on your use case. For more information, see [Connect to URL filter](/docs/apim_policydev/apigw_polref/routing_common/#connect-to-url-filter).

### Querystring passthrough per API

Select whether to enable this setting for a REST API in API Manager. When enabled, query parameters are sent unmodified to the back-end service. This is a per API implementation of the [global system property](/docs/apim_administration/apimgr_admin/api_mgmt_virtualize_web).

### Policy Studio YAML performance improvements on Windows

Policy Studio performance has been improved with focus on the YAML entity store on a Windows operating system. Waiting times have been reduced for many UI interactions.

### Install Discovery and Traceability agents

Discovery and Traceability agents can be installed using API Gateway installer. For more information see [Install agents](/docs/apim_installation/apigtw_install/install_agents/).

### New API Manager lightweight APIs

There are 3 new API Manager endpoints for retrieving a lightweight version of back-end APIs, front-end APIs, and Swagger feeds. The endpoints are available on the following paths:

* /api/portal/v1.4/apirepo/light
* /api/portal/v1.4/proxies/light
* /api/portal/v1.4/discovery/apis/light

The API Manager UI now uses these endpoints to improve performance. The models for each endpoint can be found in the [API Manager API v1.4 Swagger definition](https://apidocs.axway.com/swagger-ui-NEW/index.html?productname=apimanager&productversion=7.7.0&filename=api-manager-V_1_4-oas3.json).

In-memory caches have also been introduced for the lightweight front-end and back-end API endpoints. Caches can be enabled by setting the `com.axway.apimanager.api.data.cache` JMV parameter to `true`. When caches are enabled, these endpoints will respond significantly faster, along with the API Manager Gateway consuming more memory.

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update, which may impact your current installation.

### New redaction rules for API Gateway

New redaction rules have been defined for both Admin Node Manager and API Gateway instances. New API Gateway installations now have these rules enabled by default.

When upgrading existing installations, the default redaction files are automatically installed but not enabled.

To enable new default rules, the redaction files must be included in existing configurations, Therefore, you must modify the configuration files of your product manually to include the new redaction files.

{{< alert title="Note" >}}
To ensure that a redaction output is compatible with API Gateway versions older than **May 2022 update**, observe the following:

* If the tag `action` is not present in new rules, the default action will be `replace`.
* If the tag `replaceBy` is not present, redaction will replace `multipart/form-data` values by an empty string and `application/x-www-form-urlencoded` values by the string `null`.
  {{< /alert >}}

For more information on how to configure redaction and the format of new redaction rules, see [Redaction Rules](/docs/apim_administration/apigtw_admin/admin_redactors/).

### Support for Zulu OpenJDK and how to manually enable TLS algorithms

API Gateway and API Manager now support Zulu OpenJDK 1.8.0_322. This version of OpenJDK disables TLS algorithms version 1.0 and 1.1 by default, which might impact database connections, LDAP connections, and other connection types if such connections require the use of these algorithms.

The following sections describe how to enable TLS algorithms manually.

* **API Gateway and API Manager**: If you wish to enable these algorithms in your API Gateway or API Manager, add the `jdk.tls.disabledAlgorithms` Java security property to the jvm.xml file as follows, where `value` contains the desired list of disabled algorithms.

  ```xml
  <SecurityProperty name="jdk.tls.disabledAlgorithms" value="MD2, MD5, SHA1 jdkCA & usage TLSServer,RSA keySize < 1024, DSA keySize < 1024, EC keySize < 224" />
  ```
* **Policy Studio**: To enable these algorithms for Policy Studio, remove "TLSv1" and "TLSv1.1" from the `jdk.tls.disabledAlgorithms` property in the INSTALL_DIR/policystudio/jre/lib/security/java.security file.

### OpenSSL upgrade to version 3.0.3

OpenSSL has been upgraded to OpenSSL 3.0.3. The following are the major changes in API Gateway related to this upgrade:

* **Support of legacy algorithms**

    Cryptographic algorithms, such as DES, MD2, and RC2 are considered legacy, and their use is strongly discouraged. The legacy algorithms are still available in OpenSSL 3.0.3. For more     information, see [OpenSSL, Legacy algorithms](https://www.openssl.org/docs/man3.0/man7/migration_guide.html#Legacy-Algorithms).

    Legacy algorithms support is provided by the legacy library, which is delivered with API Gateway and referenced by the environmental variable `OPENSSL_MODULES`.

    The legacy cryptographic algorithms DES and RC2, used for PKCS12 creation in API Gateway, are replaced by AES256. DES and RC2 algorithms are still supported when reading PKCS12 files         encrypted with legacy algorithms.
* **Support of legacy engines**

    OpenSSL 3.0 introduced the Provider concept, which conflicts with the APIs used to support engines. These APIs are deprecated but still supported by legacy engine libraries delivered with API Gateway. The environmental variable `OPENSSL_ENGINES` is added to reference the legacy engines. For more details on legacy engines, see [OpenSSL, Support of legacy engines](https://www.openssl.org/docs/man3.0/man7/migration_guide.html#Support-of-legacy-engines).
* **OpenSSL configuration**

    OpenSSL configuration shipped with API Gateway (openssl.cnf) enables support of legacy algorithms and engines by default. Customized OpenSSL configurations should reflect this change.

{{< alert title="Note" color="primary" >}}Running API Gateway in FIPS mode is not yet supported.{{< /alert >}}

For more details on changes in OpenSSL 3.0.3, see [OpenSSL, Changelog](https://www.openssl.org/news/changelog.html#openssl-30).

### New system property to propagate API Manager security Invoke Policy generated headers

The ticket RDAPI-23601, from the [November 2021](/docs/apim_relnotes/20211130_apimgr_relnotes/#other-fixed-issues) release, added functionality to propagate the headers (http.headers) generated as part of an Inbound security Invoke Policy execution for further processing by API Manager, resulting in the original request headers being overwritten.

Now, a new Java system property, `com.axway.apimanager.securitydevice.httpheaders.propagate`, has been added to propagate the generated headers when required, and the previous functionality of propagating the request headers for further processing is reinstated as the default. For more information, see [System property changes](/docs/apim_reference/system_props/#77-may-2022).

### SAML SSO Metadata URL host verification is now required

When configuring SAML SSO in API Manager, the URL provided in the attribute `metadataUrl` of the `service-provider.xml` file must return a valid certificate with a matching host name. If the hostname does not match, the certificate is rejected.

If the IdP cannot provide a matching certificate, the metadata file can be manually downloaded and added to the `groups/group-2/instance-1/conf` folder alongside the `service-provider.xml` file. The `metadataUrl` attribute can then reference the relative file.

For example, in service-provider.xml:

```
metadataUrl="https://idpWithBadCert.com/idp_ADFS.xml"
```

The attribute will change to:

```
metadataUrl="./idp_ADFS.xml"
```

### Axway Terms and Conditions must be accepted to install API Gateway

Updated General Terms and Conditions (T&C) have been added to API Gateway. During installation in interactive mode, a dialog appears, and you must accept the T&C to proceed with the installation. In unattended mode, a new parameter (`ACCEPT_GENERAL_CONDITIONS`) has been introduced, which must be set to `yes` in order to run the installation.

For more information, see [Acceptance of General Conditions for license and subscription services](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/#acceptance-of-general-conditions-for-license-and-subscription-services).

### Cassandra 3.11.12 upgrade postponed

During development and testing of API Gateway with Cassandra 3.11.12, a critical issue was found in Cassandra, see [CASSANDRA-17581](https://issues.apache.org/jira/browse/CASSANDRA-17581). Stay tuned with Axway latest announcements to know when we are releasing API Gateway with Cassandra 3.11.13 instead.

### A fault handling policy is required to enable the global fault handler

When configuring the global fault handler in API Manager, it is now required to define and select a [fault handler policy](/docs/apim_administration/apimgr_admin/api_mgmt_custom_policies/#add-api-manager-fault-handler-policies).

### Corrections to the list of allowed HTTP methods for CORS request

[API Gateway CORS](/docs/apim_policydev/apigw_gw_instances/general_cors/) request returned a list of allowed HTTP methods that included methods of relative paths configured to be resolved using Longest Path match, where a CORS request path is a sub-path of the relative paths. Now, the list of allowed HTTP methods includes methods of the longest relative paths, which are sub-paths of a CORS request path.

### Obs-folded values are forbidden in HTTP headers

To prevent CRLF injection attacks, the names and values in HTTP headers are now checked for invalid formatting. The old HTTP format `obs-folded` is forbidden by default, but it can be enabled using the following system property:

```
-Dcom.vordel.dwe.outputObsFoldedValuesAllowed=true
```

For more information, see [System property changes](/docs/apim_reference/system_props/).

### API Analytics PDF reports no longer display chart contents

In API Analytics, the PDF reports no longer correctly display the contents of the charts. This issue has arisen due to a security upgrade of the `Highcharts.js` charting library. We are working on the fix of this functionality, to be released in a future update of API Gateway.

## Deprecated features

As part of our software development life cycle, we constantly review our API Management offering. As part of this update, the following capabilities have been deprecated

### Edge agent for API Gateway

The [Edge agent](https://docs.axway.com/bundle/subusage_en/page/deploy_the_agent.html) is removed from API Gateway, and it was replaced by the [Traceability and Discovery](https://docs.axway.com/bundle/amplify-central/page/docs/connect_manage_environ/connected_agent_common_reference/index.html) agents.

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

| Internal ID | Case ID                                                                                                                | Cve Identifier                                 | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| RDAPI-23654 | 01244778  01282979  01281803                                                                                           |                                                | **Issue**:  API Gateway traces, when set to log at DATA level, can contain passwords in the clear. **Resolution**: New redaction rules have been defined for both Admin Node Manager and the gateway instances. New API Gateway installations now have these rules enabled by default. When upgrading existing installations, the default redaction files will be automatically installed. The new default rules are not included in existing configurations, so you must modify your product's configuration files manually to include the new redaction files.</br> In order to have a redaction output compatible with API Gateway updates older than May 2022, observe the following:</br> - If the tag action is not present in new rules, the default action will be replace. </br> - If the tag replaceBy is not present, redaction will replace multipart/form-data values with an empty string and application/x-www-form-urlencoded values by the string null. |
| RDAPI-23779 | 01248155                                                                                                               |                                                | **Issue**: API Gateway is vulnerable to CRLF injections in HTTP headers. **Resolution**: HTTP headers' names and values are now checked for invalid formatting. The old HTTP format "obs-folded" is forbidden by default; it can be enabled using system property  "-Dcom.vordel.dwe.outputObsFoldedValuesAllowed=true"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| RDAPI-24024 | 01343778  01331493  01282085  01343977  01360967  01330775  01334869  01323363  01332552  01255316  01344660  01345491 | CVE-2021-2161 - CVE-2021-2163 - CVE-2021-35560 | **Issue**: Openjdk version 1.8.0_265 is vulnerable to CVEs CVE-2021-2161, CVE-2021-2163 and CVE-2021-35560. **Resolution**: Openjdk is updated to 1.8.0_322 which is not vulnerable to these CVEs. Note that this Openjdk version disables TLS versions 1.0 and 1.1 by default.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-26925 | 01363752                                                                                                               | CVE-2020-36518                                 | **Issue**: jackson-databind version 2.13.0 is vulnerable to CVE-2020-36518. **Resolution**: jackson-databind dependency is updated to 2.13.2.2 which is not vulnerable to this CVE.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| RDAPI-26989 | 01351991                                                                                                               | CVE-2018-20801                                 | **Issue**: API Gateway HighCharts versions 2.2.5/2.3.3 are vulnerable to CVE-2018-20801. **Resolution**: HighCharts version is now upgraded to 10.0.0, which is not vulnerable to CVE-2018-20801.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| RDAPI-27018 | 01354134  01355973                                                                                                     | CVE-2018-25032                                 | **Issue**: API Gateway zlib version 1.2.5 is vulnerable to CVE-2018-25032. **Resolution**: Zlib version is now upgraded to v1.2.12 which is not vulnerable to CVE-2018-25032.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| RDAPI-27284 | 01364606                                                                                                               |                                                | **Issue**: Retrieve from URL function in Frontend API is allowed when the domain resolves to a local loopback address. **Resolution**: Retrieve from URL function in Frontend API is now correctly validating the URL when the domain resolves to a local loopback address.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

### Other fixed issues

| Internal ID | Case ID                                          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------- | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RDAPI-19436 | 01141486                                         | **Issue**: When an API Access on the organization level is disabled, the UI is not reflecting this change in the API Access, on the Application page. **Resolution**: When API Access is disabled on the organization level, API Access toggle in the Application page is greyed-out with no change of its state. A tooltip is added to the greyed-out toggle with the description "Organization API access for this API is disabled.".                                                                                                                                                                                                                                                                                                                                     |
| RDAPI-20594 | 01162482                                         | **Issue**: When revoking an OAuth token in specific circumstances, for example, using a valid client-id and client-secret with a valid token, but the token doesn't belong to the client, and authenticating via the Authorization header, the response returns a 400 error instead of a 401 error. **Resolution**: This use case now returns 401, following the accepted OAuth RFC.                                                                                                                                                                                                                                                                                                                                                                                        |
| RDAPI-22333 | 01176021  01229877  01207140                     | **Issue**: When an OS3 definition without $.servers was imported via file into API Manager, an ambiguous error message was displayed in the API Manager UI, indicating that the OAS3 definition was malformed. **Resolution**: When an OS3 definition without $.servers is imported via file into API Manager, a more descriptive error message is now displayed: "Relative server URL not supported for scheme: 'file'. Please add $.servers to the OAS3 document, or host it as is, and import via URL."                                                                                                                                                                                                                                                                  |
| RDAPI-22760 | 01206976  01213913                               | **Issue**: APIGW_AUDIT_LOG_NAME and APIGW_AUDIT_LOG_DIR were available as configurable environment variables used in the API Gateway, but never documented. **Resolution**: Documentation has been updated to explain how to use APIGW_AUDIT_LOG_NAME and APIGW_AUDIT_LOG_DIR in an EMT environment. See, <https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_installation/apigw_containers/container_getstarted/index.html#how-do-you-customize-audit-logs>. An additional note has also been added to outline how to configure logging in a multi-node EMT environment. See, <https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_installation/apigw_containers/container_getstarted/index.html#how-do-you-customize-logs-in-a-multi-node-environment> |
| RDAPI-23379 | 01231351  01270945                               | **Issue**: When multiple API Manager traffic ports were configured - one enabled, one disabled - multiple base URLs were still appearing in the API Catalog when viewing an API, and the download of an OAS3 API sometimes contained incorrect server information. **Resolution**: It is now possible to configure multiple traffic ports - one enabled, one disabled - and have the API Catalog correctly display the base URLs. Downloading an OAS3 definition from the API Catalog now contains the correct servers object, reflecting the traffic ports that have been configured.                                                                                                                                                                                      |
| RDAPI-23471 | 01237514                                         | **Issue**: The back-end service URL for a virtualized API is not included in the export when a custom routing policy is used. **Resolution**: The back-end service URL is now included in the API collection export regardless of the chosen routing policy.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| RDAPI-23913 | 01233841                                         | **Issue**: Analytics PDF report generation is removing grid rows from the report. **Resolution**: Reports have been amended so that all grid rows are visible in the generated PDF report.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| RDAPI-23957 | 01252363                                         | **Issue**: API Manager responded with 401 status code instead of 405 status code on requests with incorrect HTTP method of an API method. **Resolution**: API Manager correctly propagates 405 responses while attempting to generate the corresponding service context.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| RDAPI-23984 | 01243317  01243812                               | **Issue**: Report UI is causing issues with Analytics/API Manager Monitoring reports when converted to GMT/UTC. **Resolution**: The UI code no longer amends the UTC/GMT time so the reports are no longer broken for particular time zones and report time combinations.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| RDAPI-24578 | 01264835                                         | **Issue**: API Manager throws NoSuchElementException if access logging is enabled and an API call with an empty cookie value is called. Nothing is written to the access log. **Resolution**: No error is thrown when access logging is enabled, and an API with an empty cookie value is called.  An empty value is logged for blank cookie value.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| RDAPI-25316 | 01314167  01230576                               | **Issue**: Save to File filter failing due to missing a proper mechanism for purging of files when the maximum number of files is reached, and many threads are invoking the filter. **Resolution**: Proper handling of exception due to file purging error during multi-thread invoking of Save to File filter is put in place.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| RDAPI-26178 | 01313635                                         | **Issue**: For back-end APIs in API Manager, 'import from topology' option for WSDL APIs on gateways without API Manager configured failed. **Resolution**: The 'import from topology' feature now works as expected for non-API Manager gateways with WSDLs configured.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| RDAPI-26209 | 01302949  01300146                               | **Issue**: An API Manager user with the special character "+" in their email address is not able to reset their password when clicking the Forgot Password link on the login screen. **Resolution**: The Forgot Password link now works for emails which contain the special character "+".                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| RDAPI-26235 | 01342478  01324122  01325469  01330592  01334175 | **Issue**:  Errors in editing scripting filters with groovy scripts using particular class imports **Resolution**: Fixed embedded set of groovy support JARs included within Policy Studio's internal plugins to include previous full listing.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| RDAPI-26247 | 01324606                                         | **Issue**: API Gateway Manager Monitoring timeline for averages are incorrectly aggregated when multiple average metric types are requested at once. **Resolution**: Timelines for averages are no longer grouped for requests of multiple metric types.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| RDAPI-26253 | 01322116                                         | **Issue**: Building docker instances with 7.7-20211130 API Gateway installer and the new apigw-emt-scripts-2.3.0 is taking roughly between 5 and 10 min to complete (until 'API Gateway setup complete' is displayed). **Resolution**: Fixed the time in building docker instances, and now it is taking less than a minute, it takes around 45s.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| RDAPI-26255 | 01321537                                         | **Issue**: When viewing an API in API Manager's API Catalog, if a user enables/disables the API and navigates back to the API Catalog screen, the 'Disabled' column does not display the correct state. **Resolution**: The API Catalog screen now displays the correct state after an API has been enabled/disabled.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| RDAPI-26298 | 01353991  01321064  01325769                     | **Issue**: Fix in RDAPI-25535 introduced a breaking change for customers who were intentionally or inadvertently using API Key section of Application to store external credential keys used to verify Inbound Security policy flow. (same for other variations) **Resolution**: System variable  "com.axway.apimanager.securitydevice.authz.legacy.enabled" can be set to "true" to allow to fallback to legacy behaviour if normal authorization fails. An error trace will be logged even if the request is authorized. This is to enable customers to fix their security device stores.                                                                                                                                                                                 |
| RDAPI-26410 | 01329340  01342614                               | **Issue**: Nodemanager didn't start after rollingback the update from November 2020 release or before to January 2021 release or later. **Resolution**: Nodemanager is now successfully started after the update is rolledback.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| RDAPI-26433 | 01327866                                         | **Issue**: The Validate Server Certificate Store Filter is persisting the Keystore Location based on the 'Check Server's java keystore' checkbox instead of the 'Check java keystore' checkbox, so the path is not being persisted and the Filter stores the passwords (for both the server & custom keystores) but after the filters first invoke, the passwords are deleted, so it can not authenticate either store on the second invoke. **Resolution**: The filter can now persist the keystore location based on what checkbox is checked. The passwords are no longer wiped after the filter gets invoked.                                                                                                                                                           |
| RDAPI-26445 | 01330497                                         | **Issue**: The 'My Credentials' page was incorrectly loading the header from the previously loaded page resulting in inconsistent options in the header. **Resolution**: This page does not have explicitly relevant header options. The header bar is now disabled in the 'My Credentials' page. All avenues of navigating to this page will now show an empty header.                                                                                                                                                                                                                                                                                                                                                                                                     |
| RDAPI-26543 | 01334364                                         | **Issue**: Some certificate subject attributes were not supported in the 'Extract Certificate Attribute' filter, resulting in 'Unknown object id - `<attribute>`' exceptions. **Resolution**: All attributes are now parsed successfully and can be viewed in the 'Extract Certificate Attribute' filter using an appropriate selector expression.                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-26585 | 01329746                                         | **Issue**: The OPTIONS request to API Gateway did not always return a correct list of allowed HTTP methods for configured longest paths. **Resolution**: The OPTIONS request to API Gateway returns a correct list of allowed HTTP methods for the corresponding Relative Path configured to be resolved using Longest Path match.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-26612 | 01230499                                         | **Issue**: Update documentation of field transaction field "finalStatus". **Resolution**: Field definition updated to: Status text of the transaction element (global policy) execution. The value is only written to the record with leg number 0 after all policies have been executed. Initialized to "null" for all records. Possible values: "Pass", "Fail", or "Error".                                                                                                                                                                                                                                                                                                                                                                                               |
| RDAPI-26618 | 01337448                                         | **Issue**: 'Extract Certificate Attributes' filter no longer extracts 'organizationIdentifier' attribute of certificates subject. **Resolution**: Extract Certificate Attributes filter now extracts 'organizationIdentifier' attribute.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| RDAPI-26688 | 01340453                                         | **Issue**: Modification of Secrets for an OAuth Credential or an API Key was not allowed, returning a 404 forbidden response when using the REST API. **Resolution**: A new system property, com.vordel.allowApiSecretModification, has been introduced. When set to 'true' in the jvm.xml file, the Secret for an OAuth Credential or an API Key can be modified via the REST API.                                                                                                                                                                                                                                                                                                                                                                                         |
| RDAPI-26726 | 01342221  01363855                               | **Issue**: API Manager UI validation on the CORS Profile Max-Age field does not allow '0' as a value, which is a valid value. **Resolution**: Validation of this field now allows '0' as a value.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| RDAPI-26763 | 01344191                                         | **Issue**: Space character trimmed from variable value in `envSettings.props` file after migration. **Resolution**: Space character is now preserved after migration.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| RDAPI-26767 | 01344377                                         | **Issue**: Users of the same group cannot run API Gateway scripts as jython classes are created with permissions 600. **Resolution**: Users of the same group can now run API Gateway scripts as jython classes are not generated.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-26810 | 01344613                                         | **Issue**: Upgrading with an empty but enabled Global Fault Handler Policy in API Manager would cause an error **Resolution**: The Global Fault Handler can now be left empty- doing so automatically defaults to the default Global Fault Handler Policy                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| RDAPI-26865 | 01346179                                         | **Issue**: API collection import fails if it contains environmentalized variables in URL fields for Security Devices. **Resolution**: URL values with environmentalized variables are now correctly validated, and API collection is successfully imported.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| RDAPI-26873 | 01326549                                         | **Issue**: logback-core-1.2.10.jar is delivered with API Gateway but is not used. **Resolution**: logback-core-1.2.10.jar is removed from API Gateway.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| RDAPI-26911 | 01341666  01357366                               | **Issue**: Organization not found error while unpublishing an API. **Resolution**: The API is unpublished now if an organization is not found. A REPORT trace is written to suggest corrective actions to be taken.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| RDAPI-26912 | 01338404                                         | **Issue**: Import configuration fragment could fail if some entities were deselected for import. **Resolution**: Import configuration fragment works as expected now when some entities are deselected for import.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-26913 | 01334214                                         | **Issue**: Incorrect WSDL document downloaded from API Catalog when several virtualized APIs from different backend APIs imported from the same URL but with different WSDL versions. **Resolution**: The correct WSDL document is now downloaded from the API Catalog.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| RDAPI-26914 | 01334214                                         | **Issue**: WSDL based API, Try It CORS from API Manager UI failing due to trailing slash. **Resolution**: Try It CORS is now successful in preserving the trailing slash as defined in the virtualized API.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| RDAPI-26920 | 01336781                                         | Previously, the Client Application Registry /availablescopes api  call was slow with a large amount of OAuth validation filters. This caused performance degradation in the web UI.Now the API is more responsive, and the UI loads quickly.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| RDAPI-27019 | 01350571  01369276                               | **Issue**: RDAPI-23601 added functionality to propagate the headers (http.headers) generated as part of an Inbound security Invoke Policy execution for further processing by API Manager, resulting in the original request headers being overwritten. **Resolution**: A new Java system property, com.axway.apimanager.securitydevice.httpheaders.propagate, has been added to propagate the generated headers when required, and the previous functionality of propagating the request headers for further processing is reinstated as the default. For more information, see <https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_reference/system_props/index.html>.                                                                                          |
| RDAPI-27021 | 01352521                                         | **Issue**: Some certificate subject attributes were not supported in the 'Extract Certificate Attribute' filter, resulting in 'Unknown object id - `<attribute>`' exceptions. **Resolution**: All attributes are now parsed successfully and can be viewed in the 'Extract Certificate Attribute' filter using an appropriate selector expression.                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RDAPI-27090 | 01342380                                         | **Issue**: API Manager /discovery/swagger/api/id/{id} endpoint was reading all virtualized APIs, as opposed to only one, from KPS when the `apiVersion` query parameter was supplied. **Resolution**: This endpoint only reads one virtualized API from KPS now.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| RDAPI-27118 | 01338489                                         | **Issue**: Instance process Heap memory is low to be released when executing XML to JSON filter. **Resolution**: Some memory leaks have been corrected in Servlet management and in XML to JSON filter. Transaction input streams not closed are now generating a warning trace when garbage collection happens.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| RDAPI-27269 | 01364186                                         | **Issue**: Temporary content spilled to disk not removed after transaction end. **Resolution**: Some conditions creating memory leak by not closing input streams have been corrected.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

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
