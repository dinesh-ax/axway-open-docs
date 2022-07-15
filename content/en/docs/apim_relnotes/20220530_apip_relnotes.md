---
title: API Portal 7.7 May 2022 Release Notes
linkTitle: API Portal May 2022
draft: false
weight: 95
date: 2022-03-22
description: null
---
API Portal updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

## Installation

API Portal is available as a software installation or a virtualized deployment in a Docker container. For more information, see the following options:

* If you are installing API Portal for the first time using this update, see [Install API Portal](/docs/apim_installation/apiportal_install/).
* If you are already using API Portal (7.5.x, 7.6.x, 7.7.x) and want to install this update, see [Upgrade API Portal](/docs/apim_installation/apiportal_install/upgrade_automatic/).
* If you are using API Portal 7.7.x with an applied patch and you want to install this update, you need to execute one manual step. See [Upgrade API Portal prerequisites](/docs/apim_installation/apiportal_install/upgrade_automatic/#api-portal-with-applied-patches).
* You can use the [cumulative upgrade script](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-api-portal-using-the-cumulative-upgrade-script) to upgrade directly from earlier versions (for example, 7.5.5, 7.6.2) to API Portal [7.7 November 2020](/docs/apim_relnotes/20201130_apip_relnotes/), then you must apply API Portal 7.7 **February 22** update, and then you can apply the **May 22** update package.
* See [API Portal single version upgrade](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-from-api-portal-7-6-2) to upgrade versions incrementally.

{{< alert title="Note" color="primary" >}}
We highly recommend following the update flow as outlined above to avoid any incorrect behaviour of the product.
{{< /alert >}}

### Docker containers

* To deploy API Portal in Docker containers, see [Deploy API Portal in containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).
* If you are already using API Portal in Docker containers and you want to install this update, see [Upgrade API Portal in Docker containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).

## New features and enhancements

The following new features and enhancements are available in this update.

### Support for Joomla 4

API Portal **May 22** update ships with Joomla 4, which is a major upgrade of the Joomla platform, bringing a new look and feel to the Joomla Admin Interface for API Portal. Joomla only allows upgrading to Joomla 4 from Joomla 3.10, which is the *bridge* release of Joomla between version 3 and version 4. As a result of this condition, a direct update to API Portal **May 22**, which is based on Joomla 4, should only be done from the API Portal [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/) release, which is based on Joomla 3.10.

Alternatively, if you have a previous installation of API Portal running on Joomla 3.9, you can independently upgrade your Joomla to 3.10, and after Joomla 3.10 upgrade is complete you can proceed with API Portal [upgrade procedure](/docs/apim_installation/apiportal_install/upgrade_automatic/).

### Monitoring charts

The library used to render the usage charts in the Monitoring area has been updated to the latest version to improve performance and security.

## Limitations of this update

This update has the following limitations:

* API Portal 7.7.20220530 is compatible with API Gateway and API Manager 7.7.20220530 only.
* For direct update to API Portal May 2022 (which is based on Joomla 4), we recommend you to update from the API Portal Feb 2022 (which is based on Joomla 3.10) first.
* This update is not available as a virtual appliance or as a managed service on Axway Cloud.

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update.

### Joomla 4 upgrade

API Portal May 22 update ships with Joomla 4, which is a major upgrade of the Joomla platform.

## Deprecated features

No features have been deprecated in this update.

<!-- As part of our software development life cycle we constantly review our API Management offering. As part of this update, the following capabilities have been deprecated-->

## End of support notices

PHP 7.1 and PHP 7.2 are no longer actively supported by the PHP, therefore we are removing these versions from API Portal in this update.

For more information, see [PHP Supported versions](https://www.php.net/supported-versions.php).

## Removed features

<!-- To stay current and align our offerings with customer demand and best practices, Axway might discontinue support for some capabilities. As part of this review, the following features have been removed: -->

No capabilities have been removed in this update.

## Fixed issues

This version of API Portal includes:

* Fixes from all 7.5.5, 7.6.2, and 7.7 service packs released prior to this version. For details of all the service pack fixes included, see the corresponding *SP Readme* attached to each service pack on [Axway Support](https://support.axway.com).
* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220530/ipp/100/product/545/version/3036/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

| Internal ID | Case ID            | CVE Identifier | Description                                                                                                                                                                        |
| ----------- | ------------------ | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAP-5175    | 1339244            |                | **Issue**: jQuery-migrate v1.4.1 was outdated. **Resolution**: Joomla 3.10.x was updated to Joomla 4.x in which jQuery-migrate v1.4.1 was not used.                                |
| IAP-5178    | 1339244            |                | **Issue**: MooTools library was used and was reported as vulnerable. **Resolution**: Joomla 3.10.x was updated to Joomla 4.x in which MooTools library was not used anymore.       |
| IAP-5179    | 01339244, 01363637 | CVE-2015-9251  | **Issue**: jQuery v1.12.4 was very old. **Resolution**: Joomla 3.10.x was updated to Joomla 4.x in which jQuery v1.12.4 was not used.                                              |
| IAP-5288    |                    |                | **Issue**: Some password input fields didn't have autocomplete attribute set to off. **Resolution**: All password input fields have autocomplete set to off.                       |
| IAP-5283    |                    |                | **Issue**: libretls version 3.3.4-r2 was reported as vulnerable. **Resolution**: libretls was updated to 3.3.4-r3.                                                                 |
| IAP-5282    |                    |                | **Issue**: libxml2 version 2.9.12-r2 was reported as vulnerable. **Resolution**: libxml2 was updated to 2.9.14-r0.                                                                 |
| IAP-5281    |                    |                | **Issue**: MariaDB version 10.6.4-r2 was reported as vulnerable. **Resolution**: MariaDB was updated to 10.6.8-r0.                                                                 |
| IAP-5280    |                    |                | **Issue**: OpenSSL version 1.1.1l-r7 was reported as vulnerable. **Resolution**: OpenSSL was updated to 1.1.1n-r0.                                                                 |
| IAP-5279    |                    |                | **Issue**: OpenSSL version 1.1.1l-r8 was reported as vulnerable. **Resolution**: OpenSSL was updated to 1.1.1n-r0.                                                                 |
| IAP-5274    |                    |                | **Issue**: zlib version 1.2.11-r3 was reported as vulnerable. **Resolution**: zlib was updated to 1.2.12-r0.                                                                       |
| IAP-5273    |                    |                | **Issue**: go version 1.17.7 was reported as vulnerable. **Resolution**: go was updated to 1.18.1.                                                                                 |
| IAP-5272    |                    |                | **Issue**: Apache HTTP Server 2.4.52 and earlier was reported as vulnerable. **Resolution**: Apache HTTP Server was updated to 2.4.53-r0.                                          |
| IAP-5271    |                    |                | **Issue**: busybox version 1.34.1-r3 was reported as vulnerable. **Resolution**: busybox was updated to 2.11.1-r2.                                                                 |
| IAP-5270    |                    |                | **Issue**: freetype version 2.11.0-r0 was reported as vulnerable. **Resolution**: freetype was updated to 2.11.1-r2.                                                               |
| IAP-5165    |                    |                | **Issue**: Vulnerable version 1.9.3-r0 of lz4 library is used. **Resolution**: lz4 is upgraded to version 1.9.3-r1.                                                                |
| IAP-5037    |                    |                | **Issue**: Vulnerable version 0.6.1 of json-pointer vdependency was used. **Resolution**: json-pointer was updated to a version 0.7.0 which is not vulnerable.                     |
| IAP-5166    |                    |                | **Issue**: API Portal uses marked package version 1.1.1, which was reported as vulnerable. **Resolution**: Marked package was upgraded to a newer non-vulnerable version.          |
| IAP-5347    |                    |                | **Issue**: Used pcre2 and apache2 libraries version was reported as vulnerable. **Resolution**: pcre2 and apache2 libraries was updated to pcre2 v10.40-r0 and apache2 v2.4.54-r0. |

### Other fixed issues

| Internal ID | Case ID | Description                                                                                                                                                                                                                                                                                                                                   |
| ----------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAP-5043    | 1322729 | **Issue**: WSDL definition download fails when it has references to non-existent xsd-s. **Resolution**: A JAI configuration "Download complete WSDL definition" was added to switch ON/OFF only valid WSDL file to be downloaded. By default the option is ON.                                                                                |
| IAP-5163    | 1322504 | **Issue**: The header "Location" in the API Manager login response was doing additional '/home' request to API Manager, which might cause session lost. **Resolution**: The HTTP client of API Portal is now configured to not follow location headers and the "/home" request is not executed anymore.                                       |
| IAP-5247    |         | **Issue**: Security credentials drop-down field is not updated when security method is changed on API details page. **Resolution**: Security credentials drop-down field is now populated with the security methods.                                                                                                                          |
| IAP-5250    | 1342380 | **Issue**: Multiple unnecessary API calls are executed causing significant performance impact. **Resolution**: Multiple optimizations : 1. If Elastic is enabled APIs per organization are taken from ES instead of API Manager. 2. APIs list is fetched only once no matter how many API catalogs are defined. 3. Other small optimizations. |
| IAP-5251    |         | **Issue**: Multiple requests to /discovery/swagger/api/id/{id} API Manager endpoint are made when you open API details page. **Resolution**: Only one request to /discovery/swagger/api/id/{id} API Manager endpoint is made when you open API details page.                                                                                  |
| IAP-5252    |         | **Issue**: /.dockerenv file doesn't get created with a container started with non-root user in some environments. Resolution: /.dockerenv file is added to the image. **Resolution**: /.dockerenv file is added to the image                                                                                                                  |

## Known issues

The following are known issues for this update.

### Custom properties permissions per role are not respected in some scenarios

Custom properties permissions per role will not be respected for the organization in the following scenarios:

* Permissions will not be respected for the organization the API is assigned to because respecting them will cause significant performance impact on the Try-it page. The custom properties permissions will follow the role of the user's primary organization.
* On creating an application, the App has no organization yet, so the custom properties permissions cannot be respected per role of the organization. They will follow the permissions for the role of the primary organization of the user.

Related Issue: IAP-4474

### When Multi Manager feature is configured, API Portal users are no longer able to login

After a recent bug fix in API Manager (RDAPI-20021), the `Authenticate to Master` policy is no longer working after upgrading from releases earlier than [API Portal July 2020](/docs/apim_relnotes/20200730_apip_relnotes/). To fix this, perform the following steps:

1. Open all slave managers configurations in Policy Studio, and click to **Edit** the `AuthenticateToMaster` policy.
2. Click the **Login to Master** (Connect to URL) filter, and enter `Accept: */*` for the **Request Protocol Header**.
3. Click the **Enter** key twice to create two blank lines after `Accept: */*`.

Alternatively you can take the updated `AuthenticateToMaster` policy and apply again your configurations.

Related issue: IAP-3435

### Page layout and alignment for Arabic language

Changing the API Portal language to Arabic (or any other right to left language) results in issues with page layout and alignment on the API Portal Home and Pricing pages, and some buttons are not visible. As a workaround, you can turn on the development mode in JAI. Follow these steps:

1. Log in to Joomla! Admin Interface (JAI).
2. In the JAI top navigation bar, click **Extensions > Templates**.
3. Click your template style (for example, `purity_III * Default`) to open it.
4. Click the **General** tab.
5. Change **Development Mode** to `ON`.
6. Click **Save** and click **Close** to close the template style.

Related issue: IAP-308

## Documentation

To find all available documentation for this product version:

1. Go to [Manuals on the Axway Documentation portal](https://docs.axway.com/bundle).
2. In the left pane **Filters** list, select your product or product version.

Customers with active support contracts need to log in to access restricted content.

For information on the different operating systems, databases, browsers, and thick client platforms supported by each Axway product, see [Supported Platforms](https://docs.axway.com/bundle/Axway_Products_SupportedPlatforms_allOS_en).

## Support services

The Axway Global Support team provides worldwide 24 x 7 support for customers with active support agreements.

Email [support@axway.com](mailto:support@axway.com) or visit [Axway Support](https://support.axway.com/).

See [Get help with API Gateway](/docs/apim_administration/apigtw_admin/trblshoot_get_help/) for the information that you should be prepared to provide when you contact Axway Support.
