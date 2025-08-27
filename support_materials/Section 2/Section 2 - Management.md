# Section 2 - Overview

This section is going to cover a lot about configuration profiles and other management resources like Policies, Smart Groups and others.

Configurations profiles in the Jamf Provider are slightly different to all other resources. The process will be to have an XML/PLIST file of the configuration and then use that to configure the terraform resource.

The main way of doing this would be to create the configuration profile in the Jamf UI in the first place, preferably in your sandbox tenant, use the Deployment Theory Jamf Pro GoLand SDK or the Jamf Pro API to download the unsigned plist and then use the plist to create your Terraform resource which will be used to promote the change through your production Route to Live.

In this section, you will be covering the following topics:

- [Lesson 1 - Smart Groups]()
- [Lesson 2 - Extension Attributes]()
- [Lesson 3 - Policies]()
- [Lesson 4 - Configuration Resource Creation]()
- [Lesson 5 - Computer Configurations]()
- [Lesson 6 - Mobile Device Configurations]()
- [Section 2 Review]()
