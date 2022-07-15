{
"title": "Upgrade API Portal container deployment",
  "linkTitle": "Upgrade container deployment",
  "weight": "30",
  "date": "2021-01-05",
  "description": "Upgrade your API Portal container deployment to a newer version."
}

## General upgrade

To upgrade your API Portal container deployment, perform the following:

1. Always back up your database and data from volumes before an upgrade.
2. Obtain a newer API Portal Docker image, available from [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/API%20Portal%20/ipp/10/product/545/version/3036/subtype/89).
3. Remove the running container:

    ```
    docker container rm -f <old-container-name>
    ```

4. Recreate a new container with the same docker run parameters as the old one:

    ```
    docker container run <old-parameters> <newer-image>
    ```

The upgrade preserves any API Portal customizations stored in volumes or database.

{{% alert title="Note" %}}
There is no support for downgrading an API Portal container deployment. Running an older API Portal container against a newer database schema will result in failure.
{{% /alert %}}

## Upgrade to API Portal May 2022 release

API Portal [May 2022](/docs/apim_relnotes/20220530_apip_relnotes/) release (7.7.20220530) is integrated with *Joomla 4*, which results in some backward incompatible changes. Attempting to upgrade directly from versions prior to [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/) to May 22 will break database integrity.

To upgrade to API Portal May 2022 release, follow these steps:

1. Download the `APIPortal_7.7.20220530_ThirdPartyPackages.zip` upgrade package from [Axway Support](https://support.axway.com).
2. If your API Portal version is lower than [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/), you must first upgrade to February 2022 as described in the previous section.
3. Create an `apiportal.ini` file with the following content:

    ```ini
    post_max_size = 60m
    upload_max_filesize = 60m
    ```

4. Restart API Portal container with your `*.ini` file mounted to the PHP configuration directory of the container:

    ```shell
    docker container rm -f <feb22-container-name>
    docker container run \
      -v <path-to-apiportal.ini-file>:/etc/php7/conf.d/apiportal.ini \
      <old-parameters> -e MYSQL_SSL_ON=0 <feb22-image>
    ```

    It is important that you use plain MySQL connection. So, you ensure the `MYSQL_SSL_ON` environment variable is set to `0`, and `MYSQL_USER` and `MYSQL_PASSWORD` environment variables contain credentials for plain connection. Also, you might need to [create a MySQL user account without TLS authentication](/docs/apim_installation/apiportal_install/install_software_configure_database/#configure-a-user-account-without-authentication).
5. With a newly started container, log in to the Joomla! Administrator Interface (JAI).
6. Click **Extensions > Plugins**, then search and disable the *T3 Framework* plugin.
7. Click **Components > Joomla! Update > Upload & Update**, then apply *Joomla 4* by uploading the relevant file from third party package.
8. Wait for the upgrade process to finish and log in to JAI again.
9. Click **System > Install > Extensions > Upload Package File**, then apply the *T3 System* plugin by uploading the relevant file from third party package.
10. Stop the `feb22` container and start a new one from the `May22` image with the old parameters, but without the mounted `apiportal.ini` file.

    ```shell
    docker container rm -f <feb22-container-name>
    docker container run \
      <old-parameters-from-prior-to-may22-container> <may22-image>
    ```
11. (Optional) If you have used MySQL SSL connection, you can change its value back to what was there before.

## Apply a service pack or patch on a container deployment

Service packs and patches provide important security updates, fixes for known issues, and improved performance for API Portal and its components.

To apply a service pack or patch to a container deployment:

1. Download a new API Portal Docker image that includes the service pack or patch from [Axway Support](https://support.axway.com/).
2. Follow the steps in [Upgrade API Portal in Docker containers](/docs/apim_installation/apiportal_docker/upgrade_docker/).
