# Section 1 - Configuration

This guide presumes that you have basic knowledge of Terraform, understand the structure of a Terraform resource and already have the Terraform provider installed and integrated to a Jamf Pro tenant.

How you set up your environment and utilise the Terraform provider will not be covered in this guide. This guide will focus on learning how to use the Jamf Pro Terraform Provider by giving examples and exercises to build your knowledge on the specific Jamf Pro resources.

In this section we are going to discuss basic configurations of a Jamf Pro server. The Jamf Pro Terraform Provider can configure all parts of the Jamf Pro tenant, in this section we are going to cover the following objects:

- Sites
- Network Segments
- Configuring Self Service
- Deploying a Health Check Policy

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage.

## Sites

Sites are used in Jamf Pro to organise and seperate object functionality into specific areas for easier management.
A site can be created in Terraform using the following resource:

`resource "jamfpro_site" "jamf_pro_site_1" {
    name = "gd-training-london"
}`

This resource will create a 'jamfpro_site' with the unique identifier 'jamf_pro_site_1'. As a site in Jamf only requires one attribute, a name. The only attribute we are required to enter here is the name, which in this case is 'gd-training-london'.

Once you apply this run, the site will be created in the linked Jamf Pro tenant.

You can see more about Sites on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/site) webpage
