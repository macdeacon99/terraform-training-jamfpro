# Terraform & GitOps Training for Jamf Pro Management

## Course Overview

This training will guide you through managing Jamf Pro resources using Terraform and GitOps practices. By the end, you'll be able to confidently manage scripts, groups, categories, extension attributes, and API configurations through code.

---

## Table of Contents

1. [Introduction to Infrastructure as Code](#module-1-introduction-to-infrastructure-as-code-iac)
2. [Understanding Our Jamf Pro Terraform Structure](#module-2-understanding-our-jamf-pro-terraform-structure)
3. [Managing Categories](#module-3-managing-categories)
4. [Managing Scripts](#module-4-managing-scripts)
5. [Managing Computer Groups](#module-5-managing-computer-groups)
6. [Managing Extension Attributes](#module-6-managing-extension-attributes)
7. [Managing API Roles and Integrations](#module-7-managing-api-roles-and-integrations)
8. [Introduction to Git and GitHub](#module-8-introduction-to-git-and-github)
9. [Git Command Line Basics](#module-9-git-command-line-basics)
10. [Conventional Commits](#module-10-conventional-commits)
11. [Release Please and Versioning](#module-11-release-please-and-versioning)
12. [Pull Request Process](#module-12-pull-request-process)
13. [Common Workflows](#module-13-common-workflows)
14. [Best Practices and Guidelines](#module-14-best-practices-and-guidelines)
15. [Troubleshooting](#module-15-troubleshooting)
16. [Hands-On Exercises](#module-16-hands-on-exercises)
17. [Quick Reference Guide](#module-17-quick-reference-guide)
- [Appendix A: Glossary](#appendix-a-glossary)
- [Appendix B: Additional Resources](#appendix-b-additional-resources)

---

## Module 1: Introduction to Infrastructure as Code (IaC)

### What is Infrastructure as Code?

Infrastructure as Code means managing and provisioning your IT infrastructure through machine-readable definition files, rather than manual configuration through the Jamf Pro web interface.

**Benefits:**
- **Version Control**: Track every change with full history
- **Consistency**: Same configuration deployed every time
- **Collaboration**: Multiple team members can work simultaneously
- **Documentation**: The code itself documents your configuration
- **Disaster Recovery**: Quickly rebuild configurations from code

### Why Terraform?

Terraform is an open-source IaC tool that lets us define Jamf Pro resources in configuration files. Changes are reviewed, approved, and automatically deployed.

---

## Module 2: Understanding Our Jamf Pro Terraform Structure

### File Organization

Our Terraform configuration is organized by resource type:

```
terraform/connected/
├── jamfpro_categories.tf
├── jamfpro_scripts.tf
├── jamfpro_scripts_data.tf
├── jamfpro_extension_attributes.tf
├── jamfpro_smart_groups.tf
├── jamfpro_smart_groups_data.tf
├── jamfpro_static_groups.tf
├── jamfpro_static_groups_data.tf
├── jamfpro_api_roles.tf
└── jamfpro_api_integrations.tf
```

### The Pattern: Data + Resource Files

Most resources follow this pattern:
- **`*_data.tf`**: Contains the actual configuration data (names, criteria, configuration)
- **`*.tf`**: Contains the Terraform resource definitions that use the data

**Example with Categories:**

**`jamfpro_categories.tf`** (the resource):
```hcl
resource "jamfpro_category" "main" {
  for_each = local.categories
  name     = each.key
  priority = each.value
}
```

This loops through `local.categories` and creates a category for each entry.

**Configuration data** (defined in the same file):
```hcl
locals {
  categories = {
    "Browsers"         = 9
    "Security"         = 3
    "Productivity"     = 9
  }
}
```

---

## Module 3: Managing Categories

### Understanding Categories

Categories in Jamf Pro organize policies, scripts, and packages. The priority value determines display order (lower numbers appear first).

### Adding a New Category

**Step 1:** Open `jamfpro_categories.tf`

**Step 2:** Add your category to the `locals` block:

```hcl
locals {
  categories = {
    // Existing categories...
    "Engineering"         = 9
    "Security"           = 3
    
    // Your new category
    "Data Science Tools" = 9
  }
}
```

**Step 3:** Commit your change (we'll cover this in Module 9)

### Best Practices for Categories
- Use descriptive names
- Standard priority is `9` unless there's a specific reason
- Priority `1-3` for high-priority categories (Sys Admin Tools, Security)
- Priority `10` for special categories like Self Service+

---

## Module 4: Managing Scripts

### Understanding Script Configuration

Scripts are defined in two files:
1. **The actual script file** in `scripts/` directory
2. **The metadata** in `jamfpro_scripts_data.tf`

### Script Structure in `jamfpro_scripts_data.tf`

```hcl
locals {
  scripts_map = {
    "filename_in_scripts_dir.sh" = {
      name            = "Human-Readable Name in Jamf Pro"
      category_id     = jamfpro_category.main["CATEGORY NAME"].id
      info            = "Description of what this script does"
      notes           = "Author and date information"
      priority        = "AFTER"  // or "BEFORE"
      os_requirements = ""
      parameter4      = "Description for parameter 4"
      parameter5      = ""
      // ... parameters 6-11
    }
  }
}
```

### Adding a New Script

**Step 1:** Place your script file in the `scripts/` directory
- Filename should have no spaces (use underscores)
- Example: `Install_Chrome_Latest.sh`

**Step 2:** Add configuration to `jamfpro_scripts_data.tf`:

```hcl
"Install_Chrome_Latest.sh" = {
  name            = "Install - Google Chrome - Latest Version"
  category_id     = jamfpro_category.main["Browsers"].id
  info            = "Downloads and installs the latest version of Google Chrome"
  notes           = "Created by Jane Smith - 15/01/2026"
  priority        = "AFTER"
  os_requirements = ""
  parameter4      = "Installation Path (leave blank for default)"
}
```

### Script Parameters

Parameters 4-11 are available for use in policies:
- Parameters 1-3 are reserved by Jamf Pro
- Use descriptive names so others know what to pass
- Empty string (`""`) if parameter is not used

### Updating an Existing Script

**Option 1: Update the script file only**
- Modify the file in `scripts/` directory
- Metadata stays the same

**Option 2: Update metadata only**
- Modify the entry in `jamfpro_scripts_data.tf`
- Script file stays the same

**Option 3: Update both**
- Modify both the script file and its metadata entry

---

## Module 5: Managing Computer Groups

### Smart Groups vs Static Groups

**Smart Groups** (`jamfpro_smart_groups_data.tf`):
- Membership automatically determined by criteria
- Examples: "All Intel Macs", "Devices with OneDrive"

**Static Groups** (`jamfpro_static_groups_data.tf`):
- Manual membership by computer ID **(not managed in Terraform)**
- Examples: "Test Devices", "VIP Users"

### Smart Group Structure

```hcl
locals {
  smart_groups = {
    "Group Name" = {
      name    = "Group Name"
      site_id = -1   // -1 means no site assignment
      criteria = [
        {
          name          = "Application Title"
          priority      = 0
          and_or        = "and"
          search_type   = "has"
          value         = "Microsoft Excel.app"
          opening_paren = false
          closing_paren = false
        }
      ]
    }
  }
}
```

### Adding a Smart Group

```hcl
"Devices without Teams" = {
  name    = "Devices without Teams"
  site_id = -1
  criteria = [
    {
      name          = "Application Title"
      priority      = 0
      and_or        = "and"
      search_type   = "does not have"
      value         = "Microsoft Teams.app"
      opening_paren = false
      closing_paren = false
    },
    {
      name          = "Last Check-in"
      priority      = 1
      and_or        = "and"
      search_type   = "less than x days ago"
      value         = "30"
      opening_paren = false
      closing_paren = false
    }
  ]
}
```

### Common Search Types
- `is` / `is not`
- `like` / `not like`
- `has` / `does not have`
- `greater than` / `less than`
- `matches regex` / `does not match regex`

### Static Group Structure

```hcl
locals {
  static_groups = [
    <!-- Just add another name of a group in this list -->
    "name-of-static-group",
    "another-group"
  ]
}
```

---

## Module 6: Managing Extension Attributes

### Understanding Extension Attributes

Extension Attributes extend Jamf Pro's inventory data with custom information. There are two types:

1. **Script-based**: Runs a script to collect data
2. **LDAP-based**: Pulls data from Azure AD/Entra ID

### Script-Based Extension Attributes

Located in `jamfpro_extension_attributes.tf`:

```hcl
locals {
  extension_attributes_scripts = {
    "Attribute Name" = {
      id                     = 123
      enabled                = true
      description            = "What this attribute collects"
      input_type             = "SCRIPT"
      script_contents        = "${path.module}/scripts/ea/script_name.sh"
      inventory_display_type = "EXTENSION_ATTRIBUTES"
      data_type              = "STRING"  // or "INTEGER"
    }
  }
}
```

### LDAP-Based Extension Attributes

```hcl
locals {
  extension_attributes_ldap = {
    "Entra ID - UserGroups" = {
      id                      = 73
      enabled                 = true
      description             = "Collect User Groups from Entra ID"
      input_type              = "DIRECTORY_SERVICE_ATTRIBUTE_MAPPING"
      ldap_attribute_mapping  = "memberOf.displayName"
      inventory_display_type  = "USER_AND_LOCATION"
      data_type               = "STRING"
      allowed_multiple_values = true
    }
  }
}
```

### Inventory Display Types
- `EXTENSION_ATTRIBUTES`: General section
- `USER_AND_LOCATION`: User-specific information
- `HARDWARE`: Hardware-related data
- `OPERATING_SYSTEM`: OS information

### Data Types
- `STRING`: Text values
- `INTEGER`: Whole numbers
- `DATE`: Date values

### Adding a New Script-Based Extension Attribute

**Step 1:** Create your script in `scripts/ea/` directory

**Step 2:** Add configuration:

```hcl
"Available Disk Space GB" = {
  enabled                = true
  description            = "Reports available disk space in GB"
  input_type             = "SCRIPT"
  script_contents        = "${path.module}/scripts/ea/Available Disk Space.sh"
  inventory_display_type = "EXTENSION_ATTRIBUTES"
  data_type              = "INTEGER"
}
```

---

## Module 7: Managing API Roles and Integrations

### Understanding API Access

API Roles and Integrations control how external systems access Jamf Pro:
- **API Roles**: Define what privileges/permissions are available
- **API Integrations**: Applications that use those roles

### API Role Structure

Located in `jamfpro_api_roles.tf`:

```hcl
locals {
  api_roles = {
    "Role Name" = {
      privileges = [
        "Read Computers",
        "Update Computers",
        "Read Scripts"
      ]
    }
  }
}
```

### Common Privileges
- `Read [Resource]`: View information
- `Create [Resource]`: Create new items
- `Update [Resource]`: Modify existing items
- `Delete [Resource]`: Remove items

### API Integration Structure

Located in `jamfpro_api_integrations.tf`:

```hcl
locals {
  api_integrations = {
    "Integration Name" = {
      enabled         = true
      access_lifetime = 3600  // seconds
      scope = [
        jamfpro_api_role.main["Role Name"]
      ]
    }
  }
}
```

### Creating a New API Integration

**Step 1:** Create the API Role with required privileges:

```hcl
"ServiceNow Read-Only" = {
  privileges = [
    "Read Computers",
    "Read Computer Inventory Collection",
    "Read Mac Applications"
  ]
}
```

**Step 2:** Create the API Integration:

```hcl
"servicenow-integration" = {
  enabled         = true
  access_lifetime = 3600
  scope = [
    jamfpro_api_role.main["ServiceNow Read-Only"]
  ]
}
```

### Access Token Lifetime
- Measured in seconds
- `60` = 1 minute
- `3600` = 1 hour (common for automated processes)
- `300` = 5 minutes (for short-lived operations)

---

## Module 8: Introduction to Git and GitHub

### What is Git?

Git is a version control system that tracks changes to files. Think of it as "track changes" for code, but much more powerful.

**Key Concepts:**
- **Repository (repo)**: A project folder with Git tracking
- **Commit**: A snapshot of changes with a description
- **Branch**: A parallel version of the code for working on features
- **Remote**: The version stored on GitHub (online)

### What is GitHub?

GitHub is a cloud platform that hosts Git repositories and provides collaboration features:
- Central storage for code
- Pull Requests for reviewing changes
- Issue tracking
- Team collaboration tools

### Our Workflow

```
Your Computer          →    GitHub
(Local Repository)          (Remote Repository)

1. Make changes
2. Commit changes       →   3. Push to branch
                            4. Create Pull Request
                            5. Review & Approve
                            6. Merge to main
                            7. Automatic deployment
```

---

## Module 9: Git Command Line Basics

### Setting Up

**First time setup (done once):**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@lloydsbanking.com"
```

There are other steps needed for using GitHub EU:

1. Sign In to GHEU using Git Credential Manager or GitHub Pull Request VS Code extension
2. Set up signed commits
3. Clone Jamf Terraform repo

### Essential Git Commands

#### 1. Checking Status

See what files have changed:
```bash
git status
```

Example output:
```
On branch main
Changes not staged for commit:
  modified:   jamfpro_scripts_data.tf
  
Untracked files:
  scripts/New_Script.sh
```

#### 2. Viewing Changes

See exactly what changed:
```bash
git diff
```

#### 3. Creating a Branch

Always work on a branch, never directly on `main`:
```bash
git checkout -b feat/add-chrome-script
```

Branch naming convention:
- `feat/description` - New features
- `fix/description` - Bug fixes
- `docs/description` - Documentation updates
- `chore/description` - Tidy up etc.

#### 4. Adding Changes

Add specific files:
```bash
git add jamfpro_scripts_data.tf
git add scripts/Install_Chrome_Latest.sh
```

Or add all changes:
```bash
git add .
```

#### 5. Committing Changes

Commit with a message (we'll cover message format in Module 10):
```bash
git commit -m "feat: add Chrome installation script"
```

#### 6. Pushing to GitHub

First push of a new branch:
```bash
git push -u origin feature/add-chrome-script
```

Subsequent pushes:
```bash
git push
```

### Complete Example Workflow

```bash
# 1. Create a branch for your work
git checkout -b feature/add-teams-fix

# 2. Make your changes to files
# (edit files in your editor)

# 3. Check what changed
git status
git diff

# 4. Stage your changes
git add jamfpro_scripts_data.tf
git add scripts/Teams_Reset.sh

# 5. Commit with a message
git commit -m "feat: add Teams reset script for high CPU issue"

# 6. Push to GitHub
git push -u origin feature/add-teams-fix

# 7. Go to GitHub and create a Pull Request
```

---

## Module 10: Conventional Commits

### What are Conventional Commits?

A standardized format for commit messages that:
- Makes the git history readable
- Enables automatic changelog generation
- Triggers appropriate version bumps

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

- **feat**: New feature (triggers MINOR version bump)
- **fix**: Bug fix (triggers PATCH version bump)
- **docs**: Documentation changes
- **chore**: Maintenance tasks (dependencies, etc.)

### Examples

**Adding a new script:**
```bash
git commit -m "feat(scripts): add Chrome installation script

Installs latest version of Google Chrome from official CDN
Includes verification of Team Identifier"
```

**Fixing a bug:**
```bash
git commit -m "fix(groups): correct regex for Tahoe compatible devices"
```

**Updating documentation:**
```bash
git commit -m "docs: add training guide for Terraform management"
```

**Simple changes:**
```bash
git commit -m "chore: update category priorities"
```

### Scope (Optional but Recommended)

The scope specifies what part of the codebase was changed:
- `scripts`: Changes to scripts
- `groups`: Changes to computer groups
- `categories`: Changes to categories
- `ea`: Changes to extension attributes
- `api`: Changes to API roles/integrations

---

## Module 11: Release Please and Versioning

### What is Release Please?

Release Please is an automated tool that:
1. Analyzes conventional commit messages
2. Determines the next version number
3. Creates a changelog
4. Creates a GitHub release
5. Triggers deployment

### Semantic Versioning

We use Semantic Versioning (SemVer): `MAJOR.MINOR.PATCH`

Example: `2.5.3`
- **MAJOR (2)**: Breaking changes
- **MINOR (5)**: New features (backwards compatible)
- **PATCH (3)**: Bug fixes

### How Commits Affect Versions

| Commit Type | Version Change | Example |
|------------|----------------|---------|
| `fix:` | PATCH | 1.0.0 → 1.0.1 |
| `feat:` | MINOR | 1.0.1 → 1.1.0 |
| `feat!:` or `BREAKING CHANGE:` | MAJOR | 1.1.0 → 2.0.0 |

### The Release Please Process

1. **You**: Merge PRs with conventional commits
2. **Release Please**: Creates a "Release PR" automatically
3. **Release PR**: Contains:
   - Updated version number
   - Changelog with all changes
   - Preview of what will be released
4. **Team**: Reviews and merges the Release PR
5. **Automation**: Deploys changes to Jamf Pro

### Example Changelog (Auto-Generated)

```markdown
## [2.5.0] - 2026-01-15

### Features
* add Chrome installation script
* add Teams reset script for high CPU issue

### Bug Fixes
* correct regex for Tahoe compatible devices
* fix category priorities for Security tools

### Documentation
* add training guide for Terraform management
```

---

## Module 12: Pull Request Process

### What is a Pull Request (PR)?

A Pull Request is a request to merge your changes into the main codebase. It enables:
- Code review by team members
- Discussion of changes
- Automated testing
- Change approval before deployment

### Creating a Pull Request

**Step 1:** Push your branch to GitHub (covered in Module 9)

**Step 2:** Go to GitHub repository

**Step 3:** Click "Pull requests" → "New pull request"

**Step 4:** Select your branch

**Step 5:** Fill out the PR template:

```markdown
## Description
Brief description of what this PR does

## Changes
- Added Chrome installation script
- Updated category for browsers

## Testing
- Tested on macOS 14.5
- Verified script parameters work correctly

## Checklist
- [x] Follows conventional commit format
- [x] Script tested locally
- [x] Documentation updated
- [x] No sensitive data in code
- [x] PR Title has Jira Ticket reference
```

### PR Best Practices

**Good PR characteristics:**
- **Small and focused**: One feature or fix per PR
- **Clear title**: Matches conventional commit format
- **Good description**: Explains why, not just what
- **Self-reviewed**: You've checked your own changes first

**Example titles:**
- ✅ `feat(scripts): EDPD-9999 add Chrome installation script`
- ✅ `fix(groups): EDPD-9999 correct Intel Mac detection criteria`
- ❌ `Updated some stuff`
- ❌ `Changes`

### Reviewing Pull Requests

When reviewing others' PRs:

**What to check:**
1. **Functionality**: Does the code do what it claims?
2. **Standards**: Follows naming conventions?
3. **Security**: No hardcoded passwords or sensitive data?
4. **Documentation**: Is it clear what this does?
5. **Testing**: Has it been tested?

**How to review:**
- Click "Files changed" tab
- Add comments on specific lines
- Use "Request changes" if issues found
- Use "Approve" if looks good
- Be constructive and helpful

### PR Status Checks

Our repository runs automated checks:
- **Terraform validation**: Syntax is correct
- **Linting**: Code follows standards
- **Security scan**: No secrets committed
- **PR Title Validation**: Has conventional commits and Jira Ticket ref.
- **Terraform Plan**: Runs a TF Plan in TF Cloud to ensure it passes

All checks must pass before merging.

### Merging a Pull Request

**Who can merge:**
- PR creator (after approval)
- Team leads
- Designated approvers

**When to merge:**
- ✅ At least one approval
- ✅ All checks passing
- ✅ No unresolved comments
- ✅ No merge conflicts

**How to merge:**
1. Click "Squash and merge" (preferred)
2. Edit the commit message if needed
3. Confirm merge
4. Delete the branch (GitHub will do this automatically)

---

## Module 13: Common Workflows

### Workflow 1: Adding a New Script

```bash
# 1. Create and checkout a branch
git checkout -b feature/add-defender-fix

# 2. Create your script file
# Create: scripts/Fix_Defender_Corruption.sh

# 3. Add script metadata
# Edit: jamfpro_scripts_data.tf
# Add entry in scripts_map

# 4. Test locally (if possible)

# 5. Stage changes
git add scripts/Fix_Defender_Corruption.sh
git add jamfpro_scripts_data.tf

# 6. Commit
git commit -m "feat(scripts): add Defender corruption fix

Removes corrupted .localized files that cause Defender issues
on macOS 15+ systems"

# 7. Push
git push -u origin feature/add-defender-fix

# 8. Create Pull Request on GitHub

# 9. Request review from team member

# 10. Address any review comments

# 11. Merge when approved
```

### Workflow 2: Updating a Smart Group

```bash
# 1. Create branch
git checkout -b fix/update-tahoe-group

# 2. Edit the smart group
# Edit: jamfpro_smart_groups_data.tf

# 3. Stage and commit
git add jamfpro_smart_groups_data.tf
git commit -m "fix(groups): update Tahoe compatible devices regex

Corrects regex to properly match MacBookPro16,1-4 models"

# 4. Push and create PR
git push -u origin fix/update-tahoe-group

# 5. Follow PR process
```

### Workflow 3: Adding a Category and Script Together

```bash
# 1. Create branch
git checkout -b feature/add-monitoring-tools

# 2. Add category
# Edit: jamfpro_categories.tf
# Add: "Monitoring" = 9

# 3. Add script that uses the category
# Create: scripts/Install_Monitoring_Agent.sh
# Edit: jamfpro_scripts_data.tf
# Reference: category_id = jamfpro_category.main["Monitoring"].id

# 4. Stage all changes
git add jamfpro_categories.tf
git add jamfpro_scripts_data.tf
git add scripts/Install_Monitoring_Agent.sh

# 5. Commit
git commit -m "feat: add monitoring tools category and installation script

- Add Monitoring category with priority 9
- Add script to install monitoring agent
- Script includes parameter for agent configuration"

# 6. Push and create PR
git push -u origin feature/add-monitoring-tools
```

### Workflow 4: Handling Merge Conflicts

Sometimes your branch conflicts with changes in `main`:

```bash
# 1. Update your local main branch
git checkout main
git pull

# 2. Go back to your branch
git checkout feature/your-branch

# 3. Merge main into your branch
git merge main

# 4. Git will tell you about conflicts
# Edit conflicted files (look for <<<<<< markers)

# 5. After fixing conflicts
git add .
git commit -m "chore: resolve merge conflicts with main"

# 6. Push updated branch
git push
```

---

## Module 14: Best Practices and Guidelines

### Naming Conventions

**Scripts:**
- Use underscores, not spaces
- Be descriptive but concise
- Include action and target
- Examples:
  - ✅ `Install_Microsoft_Teams.sh`
  - ✅ `Fix_Defender_Corruption.sh`
  - ❌ `script1.sh`
  - ❌ `my new script.sh`

**Categories:**
- Use clear, business-friendly names
- Capitalize appropriately
- Examples:
  - ✅ `Microsoft Apps`
  - ✅ `Security Tools`
  - ❌ `misc`
  - ❌ `STUFF`

**Groups:**
- Prefix with `scg-` for smart groups (optional but helpful)
- Be specific about criteria
- Examples:
  - ✅ `Devices without Defender`
  - ✅ `macOS 15.0 and above`
  - ❌ `Test Group 1`

**Branches:**
- Use descriptive names with type prefix
- Lowercase with hyphens
- Examples:
  - ✅ `feature/add-chrome-script`
  - ✅ `fix/defender-detection`
  - ❌ `johns-changes`

### Code Organization

**Keep things tidy:**
- Alphabetize entries when possible
- Use consistent indentation (2 spaces) (Run `terraform fmt --recursive` to format code)
- Add comments for complex logic
- Keep related changes in the same PR

**Example of good organization:**
```hcl
locals {
  categories = {
    // Accessibility
    "Accessibility - Cognitive" = 3
    "Accessibility - Hearing"   = 3
    
    // Core Tools
    "Browsers"                  = 9
    "Collaboration"             = 9
    
    // Security
    "Microsoft Defender"        = 9
    "Security"                  = 3
  }
}
```

### Security Considerations

**Never commit:**
- Passwords or API keys
- Personal information
- Sensitive company data
- API tokens or secrets

**Always:**
- Use variables for sensitive data
- Review changes before committing
- Report any accidentally committed secrets immediately

### Testing Before Committing

**Scripts:**
- Test on a development device if possible
- Verify all parameters work
- Check error handling

**Groups:**
- Verify criteria logic
- Test with a small scope first
- Confirm member count makes sense

**Categories:**
- Ensure the name doesn't conflict with existing categories
- Verify priority makes sense for display order

### Communication

**When to notify the team:**
- Major changes affecting multiple resources
- Breaking changes
- New categories that others might use
- Changes to shared scripts

**Where to communicate:**
- PR descriptions for change context
- Comments on specific lines of code
- Team chat for urgent issues
- Documentation updates for processes

---

## Module 15: Troubleshooting

### Common Git Issues

#### Issue: "Your branch is behind 'origin/main'"

**Solution:**
```bash
git checkout main
git pull
git checkout your-branch
git merge main
```

#### Issue: "Changes not staged for commit"

**Solution:**
```bash
# You forgot to git add
git add filename
# or
git add .
```

#### Issue: "Merge conflict in file.tf"

**Solution:**
1. Open the file in your editor
2. Look for conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
3. Edit to keep the correct version
4. Remove the conflict markers
5. `git add` the file
6. `git commit`

#### Issue: "Permission denied (publickey)"

**Solution:**
- Check your SSH key is added to GitHub
- Contact IT if you need help setting up SSH access

### Common Terraform Issues

#### Issue: "Resource already exists"

**Cause:** Resource exists in Jamf Pro but not in Terraform state

**Solution:**
- Import the resource (advanced topic - ask mac@LBG team)
- Or remove from Jamf Pro and let Terraform recreate it

#### Issue: "Invalid syntax in configuration"

**Cause:** Missing comma, bracket, or quote

**Solution:**
- Check the error message for line number
- Use an editor with syntax highlighting
- Compare to working examples

#### Issue: "Category not found"

**Cause:** Referencing a category that doesn't exist yet

**Solution:**
- Ensure the category is defined in `jamfpro_categories.tf`
- Check spelling exactly matches
- Remember Terraform is case-sensitive

### Getting Help

**Resources:**
1. **This documentation** - Your first stop
2. **Mac Team chat** - Ask questions
4. **GitHub Issues** - For bugs or feature requests

**When asking for help, provide:**
- What you were trying to do
- What happened instead
- Error messages (full text)
- What you've already tried

---

## Module 16: Hands-On Exercises

### Exercise 1: Add a Simple Script

**Goal:** Add a new script to Jamf Pro via Terraform

**Steps:**
1. Create branch: `feature/add-hello-world`
2. Create script file: `scripts/Hello_World.sh`
   ```bash
   #!/bin/bash
   echo "Hello from Terraform!"
   ```
3. Add to `jamfpro_scripts_data.tf`:
   ```hcl
   "Hello_World.sh" = {
     name            = "Test - Hello World"
     category_id     = jamfpro_category.main["Sys Admin Tools"].id
     info            = "Test script for training purposes"
     notes           = "Your Name - Today's Date"
     priority        = "AFTER"
     os_requirements = ""
   }
   ```
4. Commit with message: `feat(scripts): add hello world test script`
5. Push and create PR
6. Request review

### Exercise 2: Create a New Category

**Goal:** Add a new category for organizing resources

**Steps:**
1. Create branch: `feature/add-training-category`