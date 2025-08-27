# Lesson 1 - Scoping Groups

This lesson will describe how to create Smart Group resources. Using similar techniques as in the previous section, you can create smart groups in your terraform project and then link to them in policies either by their ID in Jamf Pro or by using the resouce ID in Terraform. Using the Terraform resource ID can make the codebase dynamic and will mean that whenever you update the Smart Group, it will automatically sync with whatever the smart group is scoped to.

In this lesson, we will cover the following topics:

- [Static Computer & Mobile Device Groups]()
- [Smart Computer & Mobile Device Groups]()

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage.

## Static Computer & User Groups

Static Computer Groups in Terraform require you to know all of the Computer IDs in Jamf Pro so that you can scope the specific devices to the Group. Creating Static Computer Groups can be done by using the following resource:

```
resource "jamfpro_static_computer_group" "jamfpro_static_computer_group_001" {
  name = "Example Static Computer Group"


  # Optional Block
  site_id = 1

  # Optional: Specify computers for static groups
  assigned_computer_ids = [1, 2, 3]
}
```

The only required attribute in this resource is the `name`. Although, if you actually want to scope devices to the group, you will need to have the `assigned_computer_ids` attribute configured as well.

You can see more about Static Computer Groups on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/static_computer_group) webpage.

Mobile Device Groups are similar to the Computer Groups, except the resource block would be:

```
resource "jamfpro_static_mobile_device_group" "jamfpro_static_mobile_device_group_001" {
  name = "Example Mobile Device Group"


  # Optional Block
  site_id = 1

  # Optional: Specify computers for static groups
  assigned_mobile_device_ids = [1, 2, 3]
}
```

You can see more about Static Computer Groups on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/static_mobile_device_group) webpage.

## Smart Computer & User Groups
