# Lesson 1 - Infrastructure

In this lesson we are going to discuss basic configuration of a Jamf Pro server. The Jamf Pro Terraform Provider can configure all parts of the Jamf Pro tenant, in this section we are going to cover the following objects:

- [Sites](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%201%20-%20Infrastructure/Lesson%201%20-%20Infrastructure.md#sites)
- [Network Segments](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%201%20-%20Infrastructure/Lesson%201%20-%20Infrastructure.md#network-segments)
- [Configuring Self Service](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%201%20-%20Infrastructure/Lesson%201%20-%20Infrastructure.md#configuring-self-service)
- [Deploying a Health Check Policy](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%201%20-%20Infrastructure/Lesson%201%20-%20Infrastructure.md#configuring-self-service)

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage.

## Sites

Sites are used in Jamf Pro to organise and seperate object functionality into specific areas for easier management.
A site can be created in Terraform using the following resource:

```
resource "jamfpro_site" "jamf_pro_site_1" {
    name = "gd-training-london"
}
```

This resource will create a 'jamfpro_site' with the unique identifier 'jamf_pro_site_1'. As a site in Jamf only requires one attribute, a name. The only attribute we are required to enter here is the name, which in this case is 'gd-training-london'.

Once you apply this run, the site will be created in the linked Jamf Pro tenant.

You can see more about Sites on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/site) webpage.

### Sites - Exercise 1 - Creating a Site

In this exercise, you will create the above site within your terraform project and push the change to your Jamf Pro instance.

Follow these steps:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `sites.tf`
2. Add in the site resource mentioned above and create a site. You can change the name attribute to anything you would like.
3. Save the file
4. Run your terraform and create the site by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory

### Sites - Exercise 2 - Creating more Sites

In this exercise, you are going to create more sites without instruction. It is worth noting here, you do not need to create a file for each site, you can create multiple resources in one terraform file. Following the steps above (excluding the file creation) and the resources provided, create 4 more sites as follows:

1. Create a site for Manchester
2. Create a site for Glasgow
3. Create a site for California
4. Create a site for Texas

## Network Segments

Network Segments in Jamf are used to limit scoping to a specific subnet of a network. A network segment resource can be created using the following format:

```
resource "jamfpro_network_segment" "jamfpro_network_segment_001" {
  name                 = "Example Network Segment"
  starting_address     = "10.16.123.0"
  ending_address       = "10.16.123.254"
  distribution_server  = "Example Distribution Server"
  distribution_point   = "Example Distribution Point"
  url                  = "http://example.com"
  swu_server           = "Example SWU Server"
  building             = "Main Building"
  department           = "IT Department"
  override_buildings   = false
  override_departments = false
}
```

This resource will create a 'jamfpro_network_segment' with the unique identifier 'jamfpro_network_segment_001'. The network segment has multiple attributes that require configuring. You can see more details on the full requirements on the registry page below. The main requirements are name, starting address and ending address.

Once you apply this run, the network segment will be created in the linked Jamf Pro tenant.

You can see more about network segments on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/network_segment) webpage.

### Network Segments - Exercise 1 - Creating a Network Segment

In this exercise, you will create a network segment and push the change to your Jamf Pro server.

Follow these steps to create the network segment:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `network_segments.tf`
2. Add in the network segment resource mentioned above and create a network segment. You can fill in the attributes with any IP range and give it a name. Following the schema, the only required fields are name, starting address and ending address, only use these fields.
3. Save the file
4. Run your terraform and create the network segment by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory

## Configuring Self Service

Self Service Customisation in Jamf will allow you to modify how your Self Service application looks and feels. The Self Service Application can be customised using the following resource:

```
resource "jamfpro_self_service_settings" "example" {
  install_automatically    = true
  install_location         = "/Applications"
  user_login_level         = "Anonymous"
  allow_remember_me        = true
  use_fido2                = false
  auth_type                = "Basic"
  notifications_enabled    = true
  alert_user_approved_mdm  = true
  default_landing_page     = "HOME"
  default_home_category_id = -1
  bookmarks_name           = "Bookmarks"
}
```

This resource will create a 'jamfpro_self_service_settings' with the unique identifier 'example'. The Self Service Configuration has multiple attributes that are required. You can see more details on the full requirements on the registry page below. All of the attributes written above are required, but there are additional optional attributes which can be found in the Terraform Registry link below.

Once you apply this run, the Self Service Configurations will be configured in the linked Jamf Pro tenant.

You can see more about network segments on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/self_service_settings) webpage.

### Configuring Self Service - Exercise 1 - Customising Self Service

In this exercise, you will customise the Self Service application to your liking for your organisation.

Follow the steps below to customise the Self Service Application:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `self_service_customisation.tf`
2. Add in the Self Service Configuration resource mentioned above and customise the Self Service application to meet the requirements of your organisation. Following the schema, you can adapt the customisation as you see fit
3. Save the file
4. Run your terraform and customise self service by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory

## Deploying a Health Check Policy

Deploying a Health Check in Jamf can be useful to determine if the tenant is online. This section will describe the process of creating a script and a policy resource that will be deployed to check the status of the Jamf Pro tenant.

### Creating and uploading the script

The following script is what will be used to check the if the Jamf Pro tenant is online:

```
#!/bin/bash

result=$(curl https://JAMF_PRO_URL.jamfcloud.com/healthCheck.html)

if [[ $result == "[]" ]]; then
    echo 'Jamf Tenant is online'
else
    echo 'Jamf Tenant is offline'
fi
```

The next step would be to create a script resource in terraform to upload the script into Jamf Pro. This can be done in 2 methods, either by creating the resource with an inline script, or by uploading the script from a file. For multi-line scripts like these, it is easier to manage by using an external file. The following resource block will show how to upload the script via a file.

```
// Script example an uploaded script taken from a file path with parameters
resource "jamfpro_script" "scripts_0001" {
  name            = "tf-jamf-health-check"
  script_contents = file("support_files/scripts/Add")
  category_id     = 5
  priority        = "BEFORE"
  info            = "Adds target user or group to specified group membership, or removes said membership."
  notes           = "Jamf Pro script parameters 4 -> 7"
}
```

This resource will create a Script object in Jamf Pro with the name `tf-jamf-health-check`, with the contents of the file being uploaded from `support_files/scripts/Add`, the Priority of `Before` and some other attributes that can be abstracted from the resource above. The required fields in this resource are the name, script contents and priority. For more information on other attributes and optional attributes you can see the Terraform Registry link below.

Once you apply this run, the script will be uploaded in the linked Jamf Pro tenant.

You can see more about scripts on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/script) webpage.

### Creating and deploying the Policy

Now that the script file is created and uploaded to Jamf, a policy can be created to deploy a policy to device via Self Service.

To create a Policy resource in Terraform, you can use the following resource block:

```
resource "jamfpro_policy" "jamfpro_policy_001" {
  name                          = "tf-jamf-health-check-001"
  enabled                       = true
  trigger_checkin               = false
  trigger_enrollment_complete   = false
  trigger_login                 = false
  trigger_network_state_changed = false
  trigger_startup               = false
  trigger_other                 = "USER_INITIATED" // "USER_INITIATED" for self service trigger , "EVENT" for an event trigger
  frequency                     = "Ongoing"
  retry_event                   = "none"
  retry_attempts                = -1
  notify_on_each_failed_retry   = false
  target_drive                  = "/"
  offline                       = false
  category_id                   = -1
  site_id                       = -1

  scope {
    all_computers = false
    all_jss_users = false

    computer_ids       = [16, 20, 21]
    computer_group_ids = sort([78, 1])

    exclusions {
      computer_ids                         = [16, 20, 21]
      computer_group_ids                   = sort([118, 1])
    }
  }

  self_service {
    use_for_self_service            = true
    self_service_display_name       = "Jamf Health Check"
    install_button_text             = "Run"
    reinstall_button_text           = "Run"
    self_service_description        = "This policy will run a health check and results can be found in policy logs"
    force_users_to_view_description = false
    feature_on_main_page            = false
  }

  payloads {
    scripts {
      id          = 123
      priority    = "After"
    }
  }
}
```

This resource will create a Policy object in Jamf Pro with the name `tf-jamf-health-check-001`, with a Script payload attached. It will also scope the policy to some computers using their computer IDs and some computer groups with their IDs. There is also some exclusions using the same methods. It will also set the policy to be run via self service. The required fields in this resource are the name, script contents and priority. For more information on other attributes and optional attributes you can see the Terraform Registry link below.

The required fields for a policy resource are enabled, name, scope and payload.

Once you apply this run, the policy will be created in the linked Jamf Pro tenant.

You can see more about policies on the policy [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/policy) webpage.

### Deploying a Health Check Policy - Exercise 1

In this exercise, you will create the resources above and deploy the script to Self Service.

Follow the steps below to create the Self Service Health Check:

1. Create a new file in your project in the `/workload/terraform/jamfpro/files` directory and name it `healthCheck.sh`. If the directory doesn't exist, create it
2. Copy the Health Check script and paste it into the `healthCheck.sh` file
3. Save the file
4. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `scripts.tf`
5. Using the example of the Script resource above, create a script resource and point it to the `healthCheck.sh` file by using the following `file("files/scripts/healthCheck.sh")`
6. Run your terraform and upload your script by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory
7. Now that the script is uploaded, browse to your Jamf Pro tenant and find the ID of the script that has been uploaded
8. Once you have the scirpt ID, you can create the Policy
9. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `policies.tf`
10. Using the policy resource example above, create a policy that uses the script that was previously uploaded by pointing the script payload to the script ID you had from the previous step
11. Run your terraform and create your policy by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory
