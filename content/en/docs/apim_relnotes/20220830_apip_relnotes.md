---
title: API Portal 7.7 August 2022 Release Notes
linkTitle: API Portal August 2022
draft: false
weight: 95
date: 2022-07-11
description: null
---
API Portal updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

This release is a small update for API Portal. It contains some performance improvements and some fixes. We are continuing to work on the new user interface (UI) and custom page builder capability with a target of delivery in the November 2022 update.

## Installation

API Portal is available as a software installation or a virtualized deployment in a Docker container. For more information, see the following options:

* If you are installing API Portal for the first time using this update, see [Install API Portal](/docs/apim_installation/apiportal_install/).
* If you are already using API Portal (7.5.x, 7.6.x, 7.7.x) and want to install this update, see [Upgrade API Portal](/docs/apim_installation/apiportal_install/upgrade_automatic/).
* If you are using API Portal 7.7.x with an applied patch and you want to install this update, you need to execute one manual step. See [Upgrade API Portal prerequisites](/docs/apim_installation/apiportal_install/upgrade_automatic/#api-portal-with-applied-patches).
* You can use the [cumulative upgrade script](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-api-portal-using-the-cumulative-upgrade-script) to upgrade directly from earlier versions (for example, 7.5.5, 7.6.2) to API Portal [7.7 November 2020](/docs/apim_relnotes/20201130_apip_relnotes/), then you must apply API Portal 7.7 **February 22** update, and then you can apply the **August 22** update package.
* See [API Portal single version upgrade](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-from-api-portal-7-6-2) to upgrade versions incrementally.

{{< alert title="Note" color="primary" >}}
We highly recommend following the update flow as outlined above to avoid any incorrect behaviour of the product.
{{< /alert >}}

### Docker containers

* To deploy API Portal in Docker containers, see [Deploy API Portal in containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).
* If you are already using API Portal in Docker containers and you want to install this update, see [Upgrade API Portal in Docker containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).

## New features and enhancements

The following new features and enhancements are available in this update.

### API Manager lightweight APIs adoption in API Catalog

With this release, API Portal is using new lightweight endpoints to improve performance of the API Catalog page when **API Information Source** is set to **Summary**. For more information, see [API information source](/docs/apim_administration/apiportal_admin/customize_apicatalog_overview/#customize-source-of-api-descriptions).

The endpoints are available at:

```none
/api/portal/v1.4/discovery/apis/light
```

They were previously released in [API Manager May 22](/docs/apim_relnotes/20220530_apimgr_relnotes/#new-api-manager-lightweight-apis) update.

## Limitations of this update

This update has the following limitations:

* API Portal 7.7.20220830 is compatible with API Gateway and API Manager 7.7.20220830 only.
* To update to API Portal August 2022 (which is based on Joomla 4) from versions older than Feb 2022 we recommend you to upgrade to API Portal Feb 2022 first, then follow the [Upgrade API Portal May 2022 or latest](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-to-api-portal-may-2022-or-latest-releases) section.
* Direct update to API Portal August 2022 can be done only from API Portal May 2022 by following the [Upgrade from the Joomla! Administrator Interface](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-from-the-joomla-administrator-interface) section.
* This update is not available as a virtual appliance or as a managed service on Axway Cloud.

## Important changes

There are no major changes in this update.

## Deprecated features

No features have been deprecated in this update.

## End of support notices

There are no end of support notices in this update

## Removed features

No capabilities have been removed in this update.

## Fixed issues

This version of API Portal includes:

* Fixes from all 7.5.5, 7.6.2, and 7.7 service packs released prior to this version. For details of all the service pack fixes included, see the corresponding *SP Readme* attached to each service pack on [Axway Support](https://support.axway.com).
* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220530/ipp/100/product/545/version/3036/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

| Internal ID | Case ID | CVE Identifier | Description                                                                                                                                                                                                                                                                       |
| ----------- | ------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAP-5190    | 1351702 |                | **Issue**: Change password form fields keep their value after modal is cancelled. **Resolution**: Change password form is reset and fields cleared when the 'Cancel' button is clicked. **Resolution**: Change password form is reset and fields cleared when the 'Cancel' button is clicked. |
| IAP-5436    |         |                | **Issue**: Vulnerable version 1.1.1o-r0 of OpenSSL was used. **Resolution**: OpenSSL was updated to version 1.1.1q-r0, which is not vulnerable.                                                                                                                                           |
| IAP-5435    |         |                | **Issue**: Vulnerable version 7.83.1-r1 of curl was used. **Resolution**: curl was updated to version 7.83.1-r2, which is not vulnerable.                                                                                                                                                 |
| IAP-5070    |         |                | **Issue**: Vulnerable version 1.25.0 of Prism was used. **Resolution**: Prism was updated to version 1.27.0, which is not vulnerable.                                                                                                                                                     |
| IAP-5485    |         | CVE-2022-37434 | **Issue**: Vulnerable version 1.2.12-r1 of Z-Library was used. **Resolution**: Z-Library was updated to version 1.2.12-r2, which is not vulnerable.                                                                                                                                       |
| IAP-5437    |         |                | **Issue**: Vulnerable version 1.35.0-r13 of BusyBox was used. **Resolution**: BusyBox was updated to version 1.35.0-r17, which is not vulnerable.                                                                                                                                         |
| IAP-5192    | 1351700 |                | **Issue**: Custom password validation rules are not enforced on the forced password change form. **Resolution**: Custom password validation rules defined in 'app-login/app.config' are enforced on password change.                                                                      |
| IAP-5354    |         |                | **Issue**: Vulnerable version 2.27.0 of Moment.js was used. **Resolution**: Moment.js was updated to version 2.29.4, which is not vulnerable.                                                                                                                                             |

### Other fixed issues

| Internal ID | Case ID | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAP-4463    | 1266708 | **Issue**: The API Manager custom property \`help\` option is not supported in API Portal. **Resolution**: Custom property \`help\` option is now supported in API Portal.                                                                                                                                                                                                                                                                                                                                                   |
| IAP-4933    | 1325688 | **Issue**: Notification message is not shown after following password reset and registration confirmation link with Apache SameSite Strict configuration. **Resolution**: Notification message is shown after following password reset and registration confirmation link.                                                                                                                                                                                                                                                   |
| IAP-5247    |         | **Issue**: Security credentials drop-down field is not updated when security method is changed on API details page. **Resolution**: Security credentials drop-down field is now populated with the security methods.                                                                                                                                                                                                                                                                                                         |
| IAP-5263    | 1367911 | **Issue**: API Portal fails showing applications and APIs from slave manager that is behind a proxy configured with lowercase headers. **Resolution**: API Portal respects lowercase headers from slave manager and now successfully shows applications and APIs.                                                                                                                                                                                                                                                            |
| IAP-5446    | 1342380 | **Issue**: API Catalog page is using an API that contains all data, which impacts the performance. **Resolution**: Now, API Catalog page has been optimized to use a lightweight endpoint to build the catalog.                                                                                                                                                                                                                                                                                                              |
| IAP-5448    | 1385517 | **Issue**: On build time, the installer being used for the build process relies on the fact that \`/.dockerenv\` file exists in Docker and that there is a process named \`docker\` in the "build-time container". This file is missing for non-docker build clients. **Resolution**: We have replaced the \`/.dockerenv\` file check and the \`docker\` process check in the API Portal installer with \`DOCKER\_ENV\` environment variable check. The environment variable is hardcoded in the Dockerfile and set to true. |
| IAP-5449    | 1386704 | **Issue**: When API Portal container is restarted or upgraded, the customized language override file gets overwritten by the one shipping with API Portal. **Resolution**: API Portal language override file gets merged with customized override language file on container restart or upgrade.                                                                                                                                                                                                                             |

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
