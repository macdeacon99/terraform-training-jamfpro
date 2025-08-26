# Lesson 3 - Enrolment

In this lesson, we are going to look at some of the Jamf Pro settings that are configured to allow users to enroll. The point of this lesson is to show how Terraform can be used to modify the Jamf Pro settings. This lesson will cover the following topics:

- [Configuring LAPS]()
- [Configuring User-initiated Enrolment]()
- [Modify LAPS Values]()

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage. You will be able to find all of the Jamf Pro settings resources on here.

## Configuring LAPS

Configuring settings is the same as configuring any other resource in Terraform. The setting that will be configured in this section will be the Local Admin Password Settings. To modify LAPS, you can use the following resource:

```
resource "jamfpro_local_admin_password_settings" "local_admin_password_settings_001" {
  auto_deploy_enabled                 = false
  password_rotation_time_seconds      = 3600
  auto_rotate_enabled                 = false
  auto_rotate_expiration_time_seconds = 7776000
}
```

There are no required attributes for this resource, only what is configured will be modified.

You can see more about API Roles on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/local_admin_password_settings) webpage.

## Configuring User-initiated Enrolment
