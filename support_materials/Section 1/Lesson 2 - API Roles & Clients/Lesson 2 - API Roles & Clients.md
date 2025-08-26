# Lesson 2 - API Roles & Clients

In this lesson, we are going to look at the basics of creating an API Role and Client. This is useful when working on automation and wanting to create scripts that use the Jamf Pro API endpoints. This lesson will cover the following topics:

- [API Roles](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%202%20-%20API%20Roles%20%26%20Clients/Lesson%202%20-%20API%20Roles%20%26%20Clients.md#api-roles)
- [API Clients](https://github.com/macdeacon99/terraform-training-jamfpro/blob/doc-updating/support_materials/Section%201/Lesson%202%20-%20API%20Roles%20%26%20Clients/Lesson%202%20-%20API%20Roles%20%26%20Clients.md#api-clients)

For more information on any of the resources we are going to create today, you can see the full breakdown from the schema on the [Terrafrom Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources) webpage.

## API Roles

Creating API Roles can be useful for creating new scripts, automations and integrating applications. To create an API Role using Terraform, you can use the following resource:

```
resource "jamfpro_api_role" "jamfpro_api_role_001" {
display_name = "all-jamf-pro-privileges-11.12"
privileges = ["Create API Integrations",
"Create API Roles"]
}
```

The required attributes for this resource are everything. It will require the `display_name` and the `privileges` list. The privileges list will be a list of strings that represent the permissions allowed for this role. A full list of privileges can be found on the Terraform registry.

You can see more about API Roles on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/api_role) webpage.

## API Clients

Creating API Clients in Terraform is similar to creating API Roles, but this is will be used to create the Client ID and Secret that utilises the API Role. The following API Client resouce block is how it will be created using Terraform:

```
resource "jamfpro_api_integration" "jamfpro_api_integration_001" {
display_name = "tf-localtest-api-integration-001"
enabled = true
access_token_lifetime_seconds = 7200
authorization_scopes = [jamfpro_api_role.jamfpro_api_role_001.display_name]
}
```

This resouce has 3 required attributes, the `display_name`, `enabled` and `authorization_scopes`. You can find out more about this resource on the Terraform registry.

This resource block brings up an important feature of Terraform which makes it extremely flexible and intelligent. On the 'autorization_sopes' attribute, the attribute is directly pointing to another resource. In this instance, it is pointing to the API Role created in the previous section. This can make creating resources extremely dynamic and prevent you from having to modify multiple resources when you need to make a change.

You can see more about API Clients on this [Terraform Registry](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/resources/api_integration) webpage.

### API Roles - Exercise 1 - Creating API Roles

In this exercise, you will create the above Jamf Pro API Role within your terraform project and push the change to your Jamf Pro instance.

Follow these steps:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `jamfpro_api_role.tf`
2. Add in the API role resource mentioned above and create a role. You can change the name attribute to anything you would like.
3. Add in some permissions that you would like, this can be anything and you can find a list on the registry webpage.
4. Save the file
5. Run your terraform and create the site by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory

### API Clients - Exercise 2 - Creating an API Client

In this exercise, you will create the above Jamf Pro API Client while pointing to the API Role that you created in the last exercise.

Follow these steps:

1. Create a new file in your project in the `/workload/terraform/jamfpro/` directory and name it `jamfpro_api_integration.tf`
2. Add in the API integration resource mentioned above and create the API Client. You can change the name attribute to anything you would like.
3. Point the `authorization_scopes` attribute to your API role that was created in Exercise 1. This is done by pointing having the value of {name of file}.{unique name of resource}.display_name
4. Save the file
5. Run your terraform and create the site by running the `terraform apply` command in the terminal in the `/workload/terraform/jamfpro` directory
