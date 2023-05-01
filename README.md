# AzureDevOps_Project 2

# Deploying Azure Resources with ARM Templates using Azure DevOps

# Step 1: Creating a Project and Service Connection in Azure DevOps

1. Go to Your organization, you can find it in the top left corner of the Azure DevOps page.
   (To create a new organization, You may watch this hands-on video: https://youtu.be/8QLp_uyy9mY)
2. Click on "+ New project" button (You can find it on the top right corner) to create a new Azure DevOps project in Your organization.
3. Choose an appropriate project name and description.
4. Select "Git" as the version control system for your project.
5. Click on "Create" to create the project.
6. Once Your project is created, click on "Project settings" in the bottom left corner.
7. In the Project Settings menu, click on "Service connections" under "Pipelines".
8. Click on the "+ New service connection" button in the top right corner and select the service you want to connect to. For this project, select "Azure Resource Manager" as the type of service connection.
9. Select the scope type as "Subscription". A new window will open for authentication to authenticate with Azure.
10. Once authenticated, select the resource group, provide a name for the service connection, and check "Grant access to all pipelines". Then click on "Save".

That's it! Your project and service connection in Azure DevOps is now created.

Watch this hands-on video here: https://youtu.be/BFE0tYMo55A

# Step 2: Creating ARM Template and YAML pipeline files to create Azure Resource Group

## Three ARM templates and YAML pipelines files are created for creating Azure resource groups

### 1. ARM Template 1

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgLocation": {
      "type": "string",
      "defaultValue": "East US2"
    },
    "rgName": {
      "type": "string",
      "defaultValue": "myRG"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "location": "[parameters('rgLocation')]",
      "name": "[parameters('rgName')]"
    }
  ],
  "outputs": {}
}
```

a. This template is to create a single resource group in Azure.

b. `$schema`: The schema defines the JSON schema for validating the ARM template. This line tells Azure which version of the schema to use for validation.

c. `contentVersion`: Specifies the version of the ARM template itself. This helps you track and manage different versions of your templates.

d. `parameters`: This section defines input parameters that users can provide when deploying the ARM template. In this case, there are two parameters: `rgLocation` and `rgName`.
   - `rgLocation`: The location of the resource group, with a default value of "East US2".
   - `rgName`: The name of the resource group, with a default value of "myRG".

e. `resources`: This section describes the resources to be created or updated by the ARM template. In this case, a single resource of type "Microsoft.Resources/resourceGroups" is being created.
   - `type`: Specifies the type of resource, which in this case is a resource group.
   - `apiVersion`: Specifies the API version to use when creating the resource.
   - `location`: Specifies the location of the resource, which is retrieved from the `rgLocation` parameter.
   - `name`: Specifies the name of the resource, which is retrieved from the `rgName` parameter.

f. `outputs`: This section defines any values that should be returned after the ARM template deployment is completed. In this case, there are no outputs defined.

### 2. ARM Template 2

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgLocation": {
      "type": "string",
      "defaultValue": "East US2"
    },
    "rgNamePrefix": {
      "type": "string",
      "defaultValue": "myResourceGroup"
    },
    "numberOfResourceGroups": {
      "type": "int",
      "defaultValue": 2
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "location": "[parameters('rgLocation')]",
      "name": "[concat(parameters('rgNamePrefix'), copyIndex(1))]",
      "copy": {
        "name": "resourceGroupCopy",
        "count": "[parameters('numberOfResourceGroups')]"
      }
    }
  ],
  "outputs": {}
}
```

a. This template is to create more than 1 resource groups in Azure. Here the prefix name of the resource group will be the same, however, a suffix number (1/2/3...) would be added depending on the number of resource groups to be created.

b. `$schema`: The schema defines the JSON schema for validating the ARM template. This line tells Azure which version of the schema to use for validation.

c. `contentVersion`: Specifies the version of the ARM template itself. This helps you track and manage different versions of your templates.

d. `parameters`: This section defines input parameters that users can provide when deploying the ARM template. In this case, there are three parameters: `rgLocation`, `rgNamePrefix`, and `numberOfResourceGroups`.
   - `rgLocation`: The location of the resource groups, with a default value of "East US2".
   - `rgNamePrefix`: The prefix of the resource group names, with a default value of "myResourceGroup".
   - `numberOfResourceGroups`: The number of resource groups to create, with a default value of 2.

e. `resources`: This section describes the resources to be created or updated by the ARM template. In this case, a single resource of type "Microsoft.Resources/resourceGroups" is being created, but with a copy loop.
   - `type`: Specifies the type of resource, which in this case is a resource group.
   - `apiVersion`: Specifies the API version to use when creating the resource.
   - `location`: Specifies the location of the resource, which is retrieved from the `rgLocation` parameter.
   - `name`: Specifies the name of the resource, which is created by concatenating the `rgNamePrefix` parameter and the `copyIndex()` function. The `copyIndex()` function starts with 1, so the first resource group name will be "myResourceGroup1", the second will be "myResourceGroup2", and so on.
   - `copy`: The copy loop is used to create multiple instances of the resource. In this case, it creates `numberOfResourceGroups` resource groups.
      - `name`: The name of the copy loop, which is "resourceGroupCopy".
      - `count`: The number of iterations for the copy loop, which is retrieved from the `numberOfResourceGroups` parameter.

f. `outputs`: This section defines any values that should be returned after the ARM template deployment is completed. In this case, there are no outputs defined.

### 3. ARM Template 3

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgLocation": {
      "type": "string",
      "defaultValue": "East US2"
    },
    "rgNamePrefix": {
      "type": "string",
      "defaultValue": "myRG-"
    },
    "rgNameSuffixes": {
      "type": "array",
      "defaultValue": [
        "app1",
        "app2"
      ]
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "location": "[parameters('rgLocation')]",
      "name": "[concat(parameters('rgNamePrefix'), parameters('rgNameSuffixes')[copyIndex()])]",
      "copy": {
        "name": "resourceGroupCopy",
        "count": "[length(parameters('rgNameSuffixes'))]"
      }
    }
  ],
  "outputs": {}
}
```

a. This template is to create more than 1 resource groups in Azure. Here the prefix name of the resource group will be the same, however, multiple suffix names can be defined in an array depending on the number of resource groups to be created.

b. `$schema`: The schema defines the JSON schema for validating the ARM template. This line tells Azure which version of the schema to use for validation.

c. `contentVersion`: Specifies the version of the ARM template itself. This helps you track and manage different versions of your templates.

d. `parameters`: This section defines input parameters that users can provide when deploying the ARM template. In this case, there are three parameters: `rgLocation`, `rgNamePrefix`, and `rgNameSuffixes`.
   - `rgLocation`: The location of the resource groups, with a default value of "East US2".
   - `rgNamePrefix`: The prefix of the resource group names, with a default value of "myRG-".
   - `rgNameSuffixes`: An array of suffixes for the resource group names, with default values of "app1" and "app2".

e. `resources`: This section describes the resources to be created or updated by the ARM template. In this case, a single resource of type "Microsoft.Resources/resourceGroups" is being created, but with a copy loop.
   - `type`: Specifies the type of resource, which in this case is a resource group.
   - `apiVersion`: Specifies the API version to use when creating the resource.
   - `location`: Specifies the location of the resource, which is retrieved from the `rgLocation` parameter.
   - `name`: Specifies the name of the resource, which is created by concatenating the `rgNamePrefix` parameter and an element from the `rgNameSuffixes` parameter array using the `copyIndex()` function.
   - `copy`: The copy loop is used to create multiple instances of the resource. In this case, it creates resource groups for each suffix in the `rgNameSuffixes` parameter.
     - `name`: The name of the copy loop, which is "resourceGroupCopy".
     - `count`: The number of iterations for the copy loop, which is the length of the `rgNameSuffixes` parameter array.

f. `outputs`: This section defines any values that should be returned after the ARM template deployment is completed. In this case, there are no outputs defined.

### YAML Pipeline

The YAML pipeline to deploy the above ARM template consists of the following tasks:

a. `trigger`: Defines the branch or branches that will trigger the pipeline when changes are pushed. In this case, the pipeline will be triggered when changes are pushed to the 'main' branch.

b. `pool`: Specifies the agent pool that runs the pipeline. The agent pool determines the virtual machines (VMs) used for the pipeline. In this case, the VM image used is 'windows-latest', which means the pipeline will run on the latest version of a Windows VM.

c. `steps`: This section contains a list of tasks that will be executed sequentially in the pipeline. In this case, there is only one task - `AzureResourceManagerTemplateDeployment@3`.
   - `task`: Specifies the task name and version. `AzureResourceManagerTemplateDeployment@3` is a task that deploys an ARM template to create or update resources in Azure.
   - `displayName`: Specifies a human-readable name for the task. In this case, the name is 'Create Resource Group using ARM Template'.
   - `inputs`: Specifies the input parameters for the task. The inputs for this task are as follows:
     - `deploymentScope`: The scope of the deployment, which in this case is set to 'Subscription'.
     - `azureResourceManagerConnection`: The service connection that provides authorization to interact with the Azure subscription.
     - `location`: The Azure region where the resources will be deployed.
     - `templateLocation`: Specifies the location of the ARM template, which is set to 'Linked artifact', meaning the template is stored as an artifact in the pipeline.
     - `csmFile`: Specifies the path to the ARM template file.

When these ARM templates are deployed, they will create new Azure resource groups with the specified name and location.

### YAML pipeline to deploy ARM Template 1
```
trigger:
- main

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureResourceManagerTemplateDeployment@3
  displayName: 'Create Resource Group using ARM Template'
  inputs:
    deploymentScope: 'Subscription'
    azureResourceManagerConnection: '<service connection name>'
    location: 'East US2'
    templateLocation: 'Linked artifact'
    csmFile: 'ARM-Template\createsingleresourcegroup.json'
```
### YAML pipeline to deploy ARM Template 2
```
    trigger:
- main

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureResourceManagerTemplateDeployment@3
  displayName: 'Create Resource Group using ARM Template'
  inputs:
    deploymentScope: 'Subscription'
    azureResourceManagerConnection: '<service connection name>'
    location: 'East US2'
    templateLocation: 'Linked artifact'
    csmFile: 'ARM-Template\createmultipleresoucegroup.json'
```
### YAML pipeline to deploy ARM Template 3
```
trigger:
- main

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureResourceManagerTemplateDeployment@3
  displayName: 'Create Resource Group using ARM Template'
  inputs:
    deploymentScope: 'Subscription'
    azureResourceManagerConnection: '<service connection name>'
    location: 'East US2'
    templateLocation: 'Linked artifact'
    csmFile: 'ARM-Template\createmultipleresoucegroup2.json'
```

That's it! Your ARM templates and YAML file are now ready.

Watch this hands-on video here: https://youtu.be/39QUdECvFRU

# Step 3: Uploading ARM template and YAML pipeline files to Github and Importing them to Azure DevOps Repo

1. Go to the GitHub website and sign in to your account.
2. On the homepage, click on the "Create a new repository" button or go to your profile and click on the "New" button.
3. Give your repository a name and add a description (optional).
4. Choose whether you want your repository to be public or private.
5. Initialize the repository with a README file.
6. Click on the "Create repository" button.
7. On the next screen, you can upload your existing code to the repository. You can drag and drop the files or click on the "Upload files" button.
8. Once your files are uploaded, you can commit them to the repository by adding a commit message and clicking on the "Commit changes" button.
9. Now in Azure DevOps, navigate to the project (Created in Step 1) where you want to create a new repo.
10. In the left-hand menu, click on "Repos".
11. In the repository page, click on the "Import" button located in center of page in "Import a repository" section.
12. In the import dialog box, select "Git" as the repository type and under clone URL paste the URL of GitHub repository that You want to import.
13. You may need to authenticate to your GitHub account and authorize Azure DevOps to access your repositories.
14. Select the repository you want to import.
15. Click on "Import" to start the import process.
16. Wait for the import process to complete. Depending on the size of your codebase, this may take a few minutes or longer.

That's it! Your template files are now uploaded to Github and imported to Azure DevOps Repo.

Watch this hands-on video here: https://youtu.be/KXYrVZa6LbA

# Step 4: Creating Azure DevOps pipeline to deploy ARM templates to create single / multiple resource groups in Azure:

1. Navigate to your Azure DevOps project and click on "Pipelines" in the left sidebar.
2. Click on "New pipeline" to create a new pipeline.
3. Choose the source of your code. Select "Azure Repos Git" and choose your repository and branch.
4. Choose a pipeline template to start with. Select the "Existing Azure pipeline YAML file" template, select Branch and path of Your YAML file (To deploy single / multiple resource groups as created in step 2 above).
5. In the "Review your pipeline YAML" step, review the pipeline YAML file and click on Run.
6. Once pipeline is completed successfully, navigate to Azure portal to validate created Resource Groups.

That's it! Your pipeline is now executed and resource groups are create in Azure portal.

Watch the hands-on videos here: 

https://youtu.be/uYtXrXd2q1Y

https://youtu.be/jyO36Umqn5Q

https://youtu.be/189Nfm4knA4


