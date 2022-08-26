{
"title": "Use YAML in Policy Studio",
"linkTitle": "Use YAML in Policy Studio",
"weight":"100",
"date": "2022-08-22",
"description": "You can use your YAML configuration within Policy Studio to develop policies, configure API Gateway instances and listeners, and configure connections to external systems."
}

The user experience in Policy Studio with a YAML project is quite similar to that of a XML project. The main differences are related to enviromentalization support.

{{< alert title="Note" >}}
This page outlines the differences between XML and YAML from a user standpoint within Policy Studio. For more information on how to develop in Policy Studio, see [Developing in Policy Studio](/docs/apim_policydev/).
{{< /alert >}}

## XML versus YAML in Policy Studio

This section describes the main differences you might encounter when using YAML in Policy Studio.

### Environmentalization

There is a significant difference in how enviromentalization is handled in YAML compared to XML. For example:

* In YAML, the same enviromentalized variable can be used across several entities, whilst in XML there is a one-to-one mapping between environmentalized variables and entities.
* In YAML, after you create a new variable using `Ctrl+E`, you can use the new variable in other filters or resources.
* In **Environment Settings** there is a **YAML Values Settings Tree Item** instead of an **Environment Settings Tree Item**.
* The **Jump to configuration** link has a slightly different behavior when compared to XML.
* When updating an environmentalized value using `values.yaml`, or editing the value within the **YAML Values Settings Tree Item**, the newly updated value is visible from within entities that use the value in question.
* The following image shows the list of available enviromentalized variables after you enter `'{'` within the chosen field.

    ![Using '{' to retrieve already defined environmentalized variables](/Images/apim_yamles/yamles_ps_e18n_placeholder_example.PNG)

#### YAML Values Settings Tree Item

There are slight differences in how the **Environment Settings Items** are rendered when comparing to XML. The following image shows the differences:

![Difference between the Environment Settings Tree and the YAML Values Settings Tree](/Images/apim_yamles/yamles_ps_e18n_trees_comparison.PNG)

#### Jump to Configuration

**Jump to configuration** behaves differently in YAML because of the nature of YAML supporting for the one-to-many mapping for its environmentalized variable.

The following image shows the defined environnement variables in your XML project, where there is a single **Jump to configuration** link per entity.

![Single Jump to configuration link for XML projects](/Images/apim_yamles/yamles_ps_e18n_xml_jump_to_config.PNG)

The following image shows the difference in YAML, where there is a **Jump to configuration** link for each field that is environmentalized.

![Various Jump to configuration links for YAML projects](/Images/apim_yamles/yamles_ps_e18n_yaml_jump_to_config.PNG)

As shown in the following image, if the environmentalized field is referenced by more than one entity, a pop-up dialog is displayed with an option to choose which reference of the environmentalized variable to jump to.

![Pop-up dialog to choose which reference to jump to](/Images/apim_yamles/yamles_ps_e18n_many_jump_to_config_dialog.PNG)

## Unsupported functionalities

The following functionalities are not supported in YAML configurations:

* Team development.
* **Compare** and **Merge** buttons.
* YAML projects can not be split up into Policy or Environment packages, meaning environment package commands are disabled.