# Section 1 - Configuration

This guide presumes that you have basic knowledge of Terraform, understand the structure of a terraform object and already have the terraform provider running on a device. How you set up your environment and utilise the terraform project will not be covered in this guide. This is soley focused on learning how to use the Jamf Pro Terraform Provider by giving examples and exercises to build your knowledge on how you can use the provider to manage a Jamf Pro instance.

In this section we are going to discuss basic configurations of a Jamf Pro server. The Jamf Pro Terraform Provider can configure all parts of the Jamf Pro tenant, in this section we are going to cover the following objects:

- Sites
- Network Segments
- Configuring Self Service
- Deploying a Health Check Policy

## Sites

Sites are used in Jamf Pro to organise and seperate object functionality into specific areas for easier management.
A site can be created in Terraform using the following block:
`resource "jamfpro_site" "jamf_pro_site_1" {
    name = "gd-training-london"
}`
