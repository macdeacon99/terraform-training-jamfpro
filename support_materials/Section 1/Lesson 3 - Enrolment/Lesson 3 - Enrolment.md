# Lesson 3 - Enrolment

In this lesson, we are going to look at some of the Jamf Pro settings that are configured to allow users to enroll. The point of this lesson is to show how Terraform can be used to modify the Jamf Pro settings. This lesson will cover the following topics:

- [Configuring LAPS](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%203%20-%20Enrolment/Lesson%203%20-%20Enrolment.md#configuring-laps)
- [Configuring User-initiated Enrolment](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%203%20-%20Enrolment/Lesson%203%20-%20Enrolment.md#configuring-user-initiated-enrolment)

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

Another settings page that is useful to configure is the User-initialed enrolment. This can be configured using the following resource in Terraform:

```
# Configure Jamf Pro User Initiated Enrollment Settings
resource "jamfpro_user_initiated_enrollment_settings" "jamfpro_uie_settings" {
  # General Settings
  restrict_reenrollment_to_authorized_users_only  = true
  skip_certificate_installation_during_enrollment = true

  # Computer Enrollment Settings
  user_initiated_enrollment_for_computers {
    enable_user_initiated_enrollment_for_computers = true
    ensure_ssh_is_enabled                          = true
    launch_self_service_when_done                  = true
    account_driven_device_enrollment               = true

    # Managed Local Administrator Account
    managed_local_administrator_account {
      create_managed_local_administrator_account                    = true
      management_account_username                                   = "jamfadmin"
      hide_managed_local_administrator_account                      = true
      allow_ssh_access_for_managed_local_administrator_account_only = true
    }
  }

  # Enrollment Messaging - English (this block is always required. It's built into the jamf gui)
  messaging {
    language_code                                   = "en"
    language_name                                   = "english"
    page_title                                      = "Welcome to Device Enrollment"
    username_text                                   = "Username"
    password_text                                   = "Password"
    login_button_text                               = "Log In"
    device_ownership_page_text                      = "Select your device type"
    personal_device_button_name                     = "Personal Device"
    institutional_ownership_button_name             = "Company Device"
    personal_device_management_description          = "Your personal device will be managed with minimal restrictions"
    institutional_device_management_description     = "This company device will be fully managed"
    enroll_device_button_name                       = "Enroll Device"
    eula_personal_devices                           = "By enrolling, you agree to allow management of your personal device"
    eula_institutional_devices                      = "This device is subject to management policies as per company guidelines"
    accept_button_text                              = "Accept"
    site_selection_text                             = "Select your site"
    ca_certificate_installation_text                = "Install CA Certificate"
    ca_certificate_name                             = "Company Root CA"
    ca_certificate_description                      = "This certificate allows secure communication with company servers"
    ca_certificate_install_button_name              = "Install CA"
    institutional_mdm_profile_installation_text     = "Install Management Profile"
    institutional_mdm_profile_name                  = "Company MDM Profile"
    institutional_mdm_profile_description           = "This profile allows management of your company device"
    institutional_mdm_profile_pending_text          = "Installing MDM profile..."
    institutional_mdm_profile_install_button_name   = "Install"
    personal_mdm_profile_installation_text          = "Install Personal Device Profile"
    personal_mdm_profile_name                       = "Personal Device Profile"
    personal_mdm_profile_description                = "Limited management profile for personal devices"
    personal_mdm_profile_install_button_name        = "Install Profile"
    user_enrollment_mdm_profile_installation_text   = "Install User Enrollment Profile"
    user_enrollment_mdm_profile_name                = "User Enrollment Profile"
    user_enrollment_mdm_profile_description         = "Profile for user-based enrollment"
    user_enrollment_mdm_profile_install_button_name = "Install"
    quickadd_package_installation_text              = "Install Management Software"
    quickadd_package_name                           = "Company MDM Agent"
    quickadd_package_progress_text                  = "Installing management software..."
    quickadd_package_install_button_name            = "Install Software"
    enrollment_complete_text                        = "Enrollment Complete! Your device is now managed."
    enrollment_failed_text                          = "Enrollment Failed. Please try again."
    try_again_button_name                           = "Try Again"
    view_enrollment_status_button_name              = "Check Status"
    view_enrollment_status_text                     = "Check your enrollment status"
    log_out_button_name                             = "Log Out"
  }

  # Directory Service Group Enrollment Settings

  # All Directory Service Users (Required when managing other directory_service_group_enrollment_settings blocks)
  # This is a special group that always exists as ID 1 and cannot be created or destroyed
  directory_service_group_enrollment_settings {
    id                                                                       = "1" // Must be provided
    allow_group_to_enroll_institutionally_owned_devices                      = true
    allow_group_to_enroll_personally_owned_devices                           = false
    allow_group_to_enroll_personal_and_institutionally_owned_devices_via_ade = true
    require_eula                                                             = true
    ldap_server_id                                                           = "-1"
    directory_service_group_name                                             = "All Directory Service Users"
    directory_service_group_id                                               = "-1"
    site_id                                                                  = "-1"
  }

  # Other groups
  directory_service_group_enrollment_settings {
    allow_group_to_enroll_institutionally_owned_devices                      = false
    allow_group_to_enroll_personally_owned_devices                           = false
    allow_group_to_enroll_personal_and_institutionally_owned_devices_via_ade = false
    require_eula                                                             = true
    ldap_server_id                                                           = data.jamfpro_cloud_idp.by_name.id // LDAP or cloud idp
    directory_service_group_name                                             = "Test M365 account"
    directory_service_group_id                                               = "27230740-e063-4931-be75-f5e9b2e4ad53"
    site_id                                                                  = "-1"
  }

  directory_service_group_enrollment_settings {
    allow_group_to_enroll_institutionally_owned_devices                      = true
    allow_group_to_enroll_personally_owned_devices                           = false
    allow_group_to_enroll_personal_and_institutionally_owned_devices_via_ade = false
    require_eula                                                             = true
    ldap_server_id                                                           = data.jamfpro_cloud_idp.by_name.id // LDAP or cloud idp
    directory_service_group_name                                             = "Test Team"
    directory_service_group_id                                               = "a2327741-8784-40bf-aa3b-7fb979ea8658"
    site_id                                                                  = "-1"
  }
}
```

There are no required attributes in this resource, but many of them are required to properly implement User-initiated enrolment. There have been a few attribute blocks removed, such as the Mobile Device enrolment. This is just to shorten this resource block for this page. A full list of attributes is the Terraform Registry page.

You can see more about API Roles on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/user_initiated_enrollment_settings) webpage.

### Enrolment - Exercise 1 - Configuring LAPS

In this exercise, you will use what you learned in the first section of this lesson and implement the LAPS configuration.

Follow these steps:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `jamfpro_local_admin_password_settings.tf`
2. Add in the resource mentioned in the LAPS section above and configure the LAPS settings.
3. Save the file
4. Run your terraform and create the site by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory

### Enrolment - Exercise 2 - Configuring User-Initiated Enrolment

In this exercise, you can customise the User-initiated enrolment as you like. There are a lot of attributes and settings in this payload, so play around with the settings and find what works best for your organisation.

Follow these steps:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `jamfpro_user_initiated_enrolment.tf`
2. Add in the resource mentioned in the enrolment section above and configure the user-initiated enrolment settings to your liking.
3. Save the file
4. Run your terraform and create the site by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory
