{
"title": "Upgrade API Portal",
  "linkTitle": "Upgrade API Portal",
  "weight": "110",
  "date": "2019-08-09",
  "description": "Upgrade your existing API Portal to 7.7."
}

You can use the [cumulative upgrade script](#upgrade-api-portal-using-the-cumulative-upgrade-script) to upgrade your 7.5.5 or 7.6.2 API Portal installation (including all service packs) directly to [7.7 November 2020](/docs/apim_relnotes/20201130_apip_relnotes/), or you can upgrade versions incrementally:

| From   | To    | Download Package         |
| ------ | ----- | -------------------------|
| 7.5.5  | [7.6.2](https://docs.axway.com/bundle/APIPortal_762_ReleaseNotes_allOS_en_HTML5/page/Content/ReleaseNotesPortal/APIPortal_ReleaseNotes_allOS_en.htm) | [7.5.5 to 7.6.2 Upgrade](https://support.axway.com/en/search/index/type/Downloads/sort/created%7Cdesc/ipp/10/product/545/version/2997/subtype/44) |
| 7.6.2  | [7.7 GA](/docs/apim_relnotes/201904_release/apip_relnotes/)                                                                                          | [7.6.2 to 7.7 Upgrade](https://support.axway.com/en/downloads/download-details/id/1443352)   |
| 7.7.x | February 22 ([7.7 20220228](/docs/apim_relnotes/20220228_apip_relnotes/)) Including all service packs                                                 | [7.7 x to 7.7 20220228](https://support.axway.com/en/search/index/type/Downloads/sort/created%7Cdesc/ipp/10/product/545/version/3036/subtype/90) |
| 7.7 20220228 |Latest                                                               | [7.7 20220228 to Latest](https://support.axway.com/en/search/index/type/Downloads/sort/created%7Cdesc/ipp/10/product/545/version/3036/subtype/90) |

{{< alert title="Note" color="primary" >}}This section does not describe how to upgrade API Gateway. For information on upgrading API Gateway, see [API Gateway Upgrade Guide](/docs/apim_installation/apigw_upgrade/).
 {{< /alert >}}

## Prerequisites

Before you upgrade your API Portal, complete the following prerequisites. These prerequisites apply for both software installation and Docker containers installation.

* If you intend to use the EasyBlog and EasyDiscuss plugins, you must install them before you start the upgrade. For more details, see [Install API Portal](/docs/apim_installation/apiportal_install/install_software/).
* Stop and back up the existing API Portal files and database. There is no option to roll back after you start the upgrade.
* Perform a back up of your API Portal software installation by taking a snapshot of your environment. Ensure that you have a file system backup and database export.

### API Portal with applied patches

If you are using API Portal with applied patches, you must change the ownership of the installation path before applying an update:

```
sudo chown -R apache:apache {ApiPortalInstallPath}
```

`{apiportalInstallPath}` is the API Portal installation directory. API Portal is installed at `/opt/axway/apiportal/htdoc` by default.

## Upgrade using the cumulative upgrade script

If you have a **7.5.5** or **7.6.2** API Portal installation, you can upgrade directly to API Portal **7.7 November 2020** by using the cumulative script.

1. Download [API Portal cumulative upgrade package](https://support.axway.com/en/search/index/type/Downloads/sort/created%7Cdesc/ipp/10/product/545/version/3036/subtype/44).
2. Change to the directory where you saved the upgrade package, and extract it:

   ```
   tar xpzvf <package_name>.tgz
   ```

3. Give executable permissions to the script `apiportal_cumulative_upgrade.sh`:

   ```
   chmod +x apiportal_cumulative_upgrade.sh
   ```

4. Execute the script:

   ```
   sh apiportal_cumulative_upgrade
   ```

## Upgrade from the Joomla! Administrator Interface

If you have a 7.7.x API Portal installation, you can upgrade to the latest version without having to repeat the initial installation setup.

1. Download the API Portal upgrade package from the [Axway Support](https://support.axway.com).
2. Change to the directory where you saved the upgrade package, and extract it:

   ```
   tar xpvzf <package_name>.tgz
   ```

3. Extract the Joomla! upgrade package (for example, `joomla-update-package-3.9.14-package.zip`) from the API Portal upgrade package to your local file system.
4. Log in to the Joomla! Administrator Interface (JAI) (`https://<API Portal host>/administrator`).
5. Click **Components > Joomla! Update**, and click the **Upload & Update** tab. If **Joomla! Update** is not visible in the menu, connect to your user database and execute the following query for API Portal database:

   ```
   update s8f7h_menu set menutype='main' where title like 'com_joomlaupdate'
   ```

6. Select the Joomla! upgrade package (for example, `joomla-update-package-3.9.14-package.zip`) from your file system.
7. Click **Upload & Install**, and follow the displayed instructions.
8. Enter the following to run the upgrade script:

   ```
   sudo ./apiportal_upgrade.sh
   ```

## Upgrade to API Portal May 2022 or latest releases

API Portal [May 2022](/docs/apim_relnotes/20220530_apip_relnotes/) release (7.7.20220530) is integrated with *Joomla 4*, which results in some backward incompatible changes. Attempting to upgrade directly from versions prior to [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/) to May 22 will break database integrity.

To upgrade to API Portal May 2022 or latest releases, follow these steps:

1. If your API Portal version is lower than [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/), you must first upgrade to February 2022 as described in the previous sections.
2. Establish an SSH connection to your API Portal server and locate the additional PHP settings directory:

    ```shell
    php --ini | grep -iF "scan for additional" | rev | cut -d' ' -f1 | rev
    ```

3. Create an `apiportal.ini` file inside the additional PHP settings directory with the following content:

    ```ini
    post_max_size = 60m
    upload_max_filesize = 60m
    ```

4. Restart the Apache server to reload the configuration. Depending on the installation type, the Apache service on your machine might have a name different from `httpd`.

    ```shell
    sudo systemctl restart httpd
    ```

5. Open API Portal in a browser, log in to the Joomla! Administrator Interface (JAI), click **System > Global Configuration > Server** and ensure that you are using `MySQLi` database driver for `Database Type` field.

    If you are using a different adapter and you need to switch to `MySQLi` adapter, ensure that you changed the database credentials accordingly. You might need to [create a MySQL user account without TLS authentication](/docs/apim_installation/apiportal_install/install_software_configure_database/#configure-a-user-account-without-authentication).
6. Click **Extensions > Plugins**, then search and disable the *T3 Framework* plugin.
7. Click **Components > Joomla! Update > Upload & Update**, then apply *Joomla 4* by uploading the relevant file from the upgrade package.
8. Wait for the upgrade process to finish.
9. Establish an SSH connection to your API Portal server and upgrade your product:

    ```shell
    sudo ./apiportal_upgrade.sh
    ```

10. (Optional) To change the `Database Type` field back to the value which was there before, in JAI, click **System > Global Configuration > Server > Database section** and change your database settings.

    Note that *Joomla 4* uses a native `MySQLi` driver in conjunction with `One-way` or `Two-way authentication` for the `Connection Encryption` field for SSL connection.

### Upgrade with mandatory prerequisites

Watch the following video to learn more about upgrading API Portal from February 2022 to May 2022 update with mandatory prerequisites:

{{< youtube id="g7Hp9pZEv4M" title="Upgrading API Portal from February 2022 to May 2022 – Mandatory Prerequisites" >}}

### Upgrade with optional prerequisites

Watch the following video to learn more about upgrading API Portal from February 2022 to May 2022 update with optional prerequisites:

{{< youtube id="gKGHw7WhPb4" title="Upgrading API Portal from February 2022 to May 2022 – Optional Prerequisites" >}}

## Post-upgrade steps

After the upgrade, perform the following tasks.

### Reinstall Joomla! components

After upgrade, you must reinstall Easyblog and EasyDiscuss in JAI to update the component version and fix compatibility issues. The API Portal data related to the components (posts, attachments) is not affected.

1. Log in to the JAI.
2. Click **Components > EasyBlog**, and follow the instructions in the EasyBlog installer.
3. If prompted to select the installation method, select **Installation via Directory**, select the available package from the drop-down list, and follow the instructions in the installer to the finish. Do not install any of the modules and plugins unless you plan to use them. To prevent installing any modules, click **Modules** and deselect **Select All**, then repeat the same for **Plugins**.
4. Click **Components > EasyDiscuss**, and repeat the component installation as described for EasyBlog.
5. If a newer version is available for **EasyBlog** or **EasyDiscuss**, click **Update Now** to update the component.

{{< alert title="Note" color="primary" >}} To resolve a known issue (caused by EasyBlog) with broken menu paths when creating new custom menus for your API Portal in JAI, you must rebuild the menu paths. In JAI, select **Menus > Main Menu** and click **Rebuild**. You only need to rebuild the menu paths once after installation or upgrade. {{< /alert >}}

### Restore footer customizations

If you customized the company name in your API Portal footer using a Joomla! language override (**Extensions > Languages > Overrides** in JAI), you must perform the following steps to restore language overrides after upgrade:

1. Locate the backup file `/opt/axway/apiportal/htdoc/Backups/<timestamp>/language/overrides/en-GB.override.ini` that is created during an upgrade. The timestamp corresponds to the time of the upgrade, for example, `20180613105149`.
2. Copy the backup file to the path `/opt/axway/apiportal/htdoc/language/overrides/en-GB.override.ini`.

Any customizations performed using language overrides are now restored.

### Hide blog from the API Portal landing page (optional)

After the upgrade, the blog is visible and accessible on the main menu on the API Portal landing page, because the EasyBlog plugin overrides permissions. If you want to hide your blog on the landing page, do the following:

1. Log in to JAI.
2. Click **Menus > Main Menu**, and select **Blog**.
3. On the **Details** tab, set **Access** to **Registered**, and click **Save**.
4. Refresh the API Portal landing page. Blog is no longer visible in the main menu before signing in.

### Consolidate vhosts and .htaccess files (optional)

During upgrade, the original `vhost` file is backed up to the following location: `/etc/httpd/conf.d/apiportal.conf.old`.

A new `vhost` file is deployed at the same location. If you had any customization, which you want to preserve, you must merge the old and new files together manually.

Similarly, the original `.htaccess` file is backed up to `${apiportal-install-dir}/.htaccess.old`. However, the two files are merged automatically.

### Encrypt database password

If you did not choose to encrypt your database password during the installation process, you can use the `apiportal_db_pass_encryption.sh` script, available from both API Portal installation and upgrade packages, to encrypt the password at any time. For more details see [Encrypt database password](/docs/apim_installation/apiportal_install/secure_harden_portal/#encrypt-database-password).

## Update API Portal with a service pack or patch

Service packs and patches provide important security updates, fixes for known issues, and improved performance for API Portal and its components.

To apply a service pack or patch on a software installation:

1. Download the service pack and the associated Readme file from [Axway Support](https://support.axway.com/).
2. Review the Readme file for any specific instructions before you apply the service pack.
