# Lesson 1 - Scoping Groups

This lesson will describe how to create Smart Group resources. Using similar techniques as in the previous section, you can create smart groups in your terraform project and then link to them in policies either by their ID in Jamf Pro or by using the resouce ID in Terraform. Using the Terraform resource ID can make the codebase dynamic and will mean that whenever you update the Smart Group, it will automatically sync with whatever the smart group is scoped to.

In this lesson, we will cover the following topics:

- [Static Computer & User Groups]()
- [Smart Computer & User Groups]()

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage.
