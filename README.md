# Migrate Public Endpoint Web App Between App Service Plans using Devops

Greetings to my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate how to __Migrate Public Endpoint Web App Between App Service Plans__.

| __AUTOMATION OBJECTIVES:-__ |
| --------- |

| __#__ | __TOPICS__ |
| --------- | --------- |
|  1. | Validate if Resource Group exists. |
|  2. | Validate if Source App Service Plan Exists. |
|  3. | Validate if Destination App Service Plan Exists. |
|  4. | Validate if App Service Exists. |
|  5. | Validate Webspace. |
|  6. | If all the above validation is successful, Web App will then be migrated to Destination App Service Plan.  |

| __IMPORTANT NOTE:-__ |
| --------- |
The YAML Pipeline is tested on __WINDOWS BUILD AGENT__ Only!!!
 
| __REQUIREMENTS:-__ |
| --------- |
1. Azure Subscription.
2. Azure DevOps Organisation and Project.
3. Service Principal with Required RBAC ( __Contributor__) applied on Subscription or Resource Group(s). 
4. Azure Resource Manager Service Connection in Azure DevOps.
5. One Demo Resource Group.
6. One Source App Service Plan.
7. One Destination App Service Plan.
8. One Web App with Public Endpoint. 

| __LIST OF AZURE RESOURCES DEPLOYED FOR THIS AUTOMATION:-__ |
| --------- |
| Azure Resources: 1) Source App Service Plan, 2) Destination App Service Plan, and 3) Web App.   |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/idn00ef2qfeb246qe96l.jpg) |
| Web App is running on Source App Service Plan. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3f8fw7xt8ca1s8gng09v.jpg) |
| There is no Web App running on Destination App Service Plan.  |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xq0euq9542fg8r3tob3x.jpg) |

| __HOW DOES MY CODE PLACEHOLDER LOOKS LIKE:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/deqqmkwxgxe8zuh5hqie.jpg) |

| PIPELINE CODE SNIPPET:- | 
| --------- |

| AZURE DEVOPS YAML PIPELINE (azure-pipelines-migrate-publicendpoint-webapp-between-AppService-Plans-v1.0.yml):- |
| --------- |

```
trigger:
  none

######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID details follows below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGName
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: AppServiceSourcePlanName
  displayName: Please Provide the Source App Service Plan Name:-
  type: object
  default: 

- name: AppServiceDestinationPlanName
  displayName: Please Provide the Destination App Service Plan Name:-
  type: object
  default:

- name: WebAppName
  displayName: Please Provide the App Service Name:-
  type: object
  default:

######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest
  envName: NonProd
  
#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: $(BuildAgent)

###################
# Declare Stages:-
###################

stages:

- stage: Validate_ResourceGroup_AppServicePlans_WebApp
  jobs:
  - job: Validate_ResourceGroup_AppServicePlans_WebApp 
    displayName: Validate Resource Group App Service Plans and WebApp
    steps:
    - task: AzureCLI@2
      displayName: Set Azure Account
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show  
    - task: AzureCLI@2
      displayName: Validate Resource Group
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $i = az group exists -n ${{ parameters.RGName }}
            if ($i -eq "true") {
              echo "#####################################################"
              echo "Resource Group ${{ parameters.RGName }} exists!!!"
              echo "#####################################################"              
            }
            else {
              echo "#############################################################"
              echo "Resource Group ${{ parameters.RGName }} DOES NOT exists!!!"
              echo "#############################################################"
              exit 1
            }
    - task: AzureCLI@2
      displayName: Validate Source App Service Plan
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $j = az appservice plan list --query "[?name=='${{ parameters.AppServiceSourcePlanName }}'].name" -o tsv
            if ($j -eq "${{ parameters.AppServiceSourcePlanName }}") {
              echo "##############################################################################"
              echo "Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} exists!!!"
              echo "##############################################################################"              
            }
            else {
               echo "#######################################################################################"
              echo "Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} DOES NOT exists!!!"
              echo "########################################################################################"
              exit 1
            }
    - task: AzureCLI@2
      displayName: Validate Destination App Service Plan
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $k = az appservice plan list --query "[?name=='${{ parameters.AppServiceDestinationPlanName }}'].name" -o tsv
            if ($k -eq "${{ parameters.AppServiceDestinationPlanName }}") {
              echo "#########################################################################################"
              echo "Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }} exists!!!"
              echo "#########################################################################################"              
            }
            else {
               echo "#################################################################################################"
              echo "Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }} DOES NOT exists!!!"
              echo "##################################################################################################"
              exit 1
            }        
    - task: AzureCLI@2
      displayName: Validate Web App
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $l = az webapp list --query "[?name=='${{ parameters.WebAppName }}'].name" -o tsv
            if ($l -eq "${{ parameters.WebAppName }}") {
              echo "#####################################################"
              echo "Web App ${{ parameters.WebAppName }} exists!!!"
              echo "#####################################################"              
            }
            else {
               echo "#############################################################"
              echo "Web App ${{ parameters.WebAppName }} DOES NOT exists!!!"
              echo "##############################################################"
              exit 1
            }
    - task: AzureCLI@2
      displayName: Validate Webspace
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $srcplanwebspace = az appservice plan show --name ${{ parameters.AppServiceSourcePlanName }} --resource-group ${{ parameters.RGName }} --query ["properties.webSpace"] -o tsv
          $destplanwebspace = az appservice plan show --name ${{ parameters.AppServiceDestinationPlanName }} --resource-group ${{ parameters.RGName }} --query ["properties.webSpace"] -o tsv
            if ($srcplanwebspace -eq $destplanwebspace) {
              echo "################################################"
              echo "Webspace of both App Service Plans matches!!!"
              echo "################################################"              
            }
            else {
              echo "##############################################################################################################################"
              echo "Webspace of both App Service Plans DOES NOT match and hence App Service ${{ parameters.WebAppName }} cannot be Migrated !!!"
              echo "##############################################################################################################################"
              exit 1
            }

- stage: Migrate_WebApp_Between_AppServicePlans
  condition: |
     and(succeeded(), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')
     )
  jobs:
  - deployment: 
    displayName: Migrate WebApp Between App Service Plans
    environment: '$(envName)'
    pool:
      vmImage: $(BuildAgent)
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            displayName: Set Azure Account
            inputs:
              azureSubscription: $(ServiceConnection)
              scriptType: ps
              scriptLocation: inlineScript
              inlineScript: |
                az --version
                az account set --subscription ${{ parameters.SubscriptionID }}
                az account show
          - task: AzureCLI@2
            displayName: Migrate WebApp 
            inputs:
              azureSubscription: $(ServiceConnection)
              scriptType: ps
              scriptLocation: inlineScript
              inlineScript: |
                az webapp update -g ${{ parameters.RGName }} -n ${{ parameters.WebAppName }} --set serverFarmId=/subscriptions/${{ parameters.SubscriptionID }}/resourceGroups/${{ parameters.RGName }}/providers/Microsoft.Web/serverfarms/${{ parameters.AppServiceDestinationPlanName }}
                echo "###########################################################################################################################################################################################################"
                echo "WebApp ${{ parameters.WebAppName }} migrated from Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} to Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }}!!!"
                echo "###########################################################################################################################################################################################################"
```

Now, let me explain each part of YAML Pipeline for better understanding.

| PART #1:- |
| --------- |

| BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:- |
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID details follows below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGName
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: AppServiceSourcePlanName
  displayName: Please Provide the Source App Service Plan Name:-
  type: object
  default: 

- name: AppServiceDestinationPlanName
  displayName: Please Provide the Destination App Service Plan Name:-
  type: object
  default:

- name: WebAppName
  displayName: Please Provide the App Service Name:-
  type: object
  default:
```

| THIS IS HOW IT LOOKS:- |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xinmeo71bfmbsj1ima6z.jpg) |

| PART #2:- |
| --------- |

| BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:- |
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest
  envName: NonProd
```

| __NOTE:-__ |
| --------- |
| Please change the values of the variables accordingly. |
| The entire YAML pipeline is build using Runtime Parameters and Variables. No Values are Hardcoded. |

| PART #3:- |
| --------- |

| This is a 2 Stage Pipeline. |
| --------- |
| Stage #1 of the Pipeline __"Validate_ResourceGroup_AppServicePlans_WebApp"__ has 6 Pipeline Tasks. |
| Stage #2 of the Pipeline __"Migrate_WebApp_Between_AppServicePlans"__ has 2 Pipeline Tasks. |

| STAGE #1 - PIPELINE TASK #1:- |
| --------- |
| Set Azure Account. |

```
- stage: Validate_ResourceGroup_AppServicePlans_WebApp
  jobs:
  - job: Validate_ResourceGroup_AppServicePlans_WebApp 
    displayName: Validate Resource Group App Service Plans and WebApp
    steps:
    - task: AzureCLI@2
      displayName: Set Azure Account
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
```
 
| STAGE #1 - PIPELINE TASK #2:- |
| --------- |
| Validate if Resource Group exists. If not, Pipeline will fail. |

```
- task: AzureCLI@2
      displayName: Validate Resource Group
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $i = az group exists -n ${{ parameters.RGName }}
            if ($i -eq "true") {
              echo "#####################################################"
              echo "Resource Group ${{ parameters.RGName }} exists!!!"
              echo "#####################################################"              
            }
            else {
              echo "#############################################################"
              echo "Resource Group ${{ parameters.RGName }} DOES NOT exists!!!"
              echo "#############################################################"
              exit 1
            }
```

| STAGE #1 - PIPELINE TASK #3:- |
| --------- |
| Validate if Source App Service Plan exists. If not, Pipeline will fail. |

```
- task: AzureCLI@2
      displayName: Validate Source App Service Plan
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $j = az appservice plan list --query "[?name=='${{ parameters.AppServiceSourcePlanName }}'].name" -o tsv
            if ($j -eq "${{ parameters.AppServiceSourcePlanName }}") {
              echo "##############################################################################"
              echo "Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} exists!!!"
              echo "##############################################################################"              
            }
            else {
               echo "#######################################################################################"
              echo "Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} DOES NOT exists!!!"
              echo "########################################################################################"
              exit 1
            }
```

| STAGE #1 - PIPELINE TASK #4:- |
| --------- |
| Validate if Destination App Service Plan exists. If not, Pipeline will fail. |

```
- task: AzureCLI@2
      displayName: Validate Destination App Service Plan
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $k = az appservice plan list --query "[?name=='${{ parameters.AppServiceDestinationPlanName }}'].name" -o tsv
            if ($k -eq "${{ parameters.AppServiceDestinationPlanName }}") {
              echo "#########################################################################################"
              echo "Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }} exists!!!"
              echo "#########################################################################################"              
            }
            else {
               echo "#################################################################################################"
              echo "Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }} DOES NOT exists!!!"
              echo "##################################################################################################"
              exit 1
            } 
```

| STAGE #1 - PIPELINE TASK #5:- |
| --------- |
| Validate if Web App exists. If not, Pipeline will fail. |

```
- task: AzureCLI@2
      displayName: Validate Web App
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $l = az webapp list --query "[?name=='${{ parameters.WebAppName }}'].name" -o tsv
            if ($l -eq "${{ parameters.WebAppName }}") {
              echo "#####################################################"
              echo "Web App ${{ parameters.WebAppName }} exists!!!"
              echo "#####################################################"              
            }
            else {
               echo "#############################################################"
              echo "Web App ${{ parameters.WebAppName }} DOES NOT exists!!!"
              echo "##############################################################"
              exit 1
            }
```

| STAGE #1 - PIPELINE TASK #6:- |
| --------- |
| Validate Webspace. If not, Pipeline will fail. |

```
- task: AzureCLI@2
      displayName: Validate Webspace
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |      
          $srcplanwebspace = az appservice plan show --name ${{ parameters.AppServiceSourcePlanName }} --resource-group ${{ parameters.RGName }} --query ["properties.webSpace"] -o tsv
          $destplanwebspace = az appservice plan show --name ${{ parameters.AppServiceDestinationPlanName }} --resource-group ${{ parameters.RGName }} --query ["properties.webSpace"] -o tsv
            if ($srcplanwebspace -eq $destplanwebspace) {
              echo "################################################"
              echo "Webspace of both App Service Plans matches!!!"
              echo "################################################"              
            }
            else {
              echo "##############################################################################################################################"
              echo "Webspace of both App Service Plans DOES NOT match and hence App Service ${{ parameters.WebAppName }} cannot be Migrated !!!"
              echo "##############################################################################################################################"
              exit 1
            }
```

| IMPORTANT TO NOTE: WHY WE NEED TO VALIDATE WEBSPACE ? |
| --------- |
| Validating webspace ensures whether we can migrate the Web App between the mentioned App Service Plans (Source and Destination).  |
| Azure deploys each new App Service plan into a deployment unit, internally called a webspace. Each region can have many webspaces, but your app can only move between plans that are created in the same webspace. |
| It is not possible to specify the webspace when we are creating a plan, but it’s possible to ensure that a plan is created in the same webspace as an existing plan. In brief, __all plans created with the same resource group, region and operating system are deployed into the same webspace__. |
| __Webspace Information of Source App Service Plan.__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pt1gr3233o4lygkk9o0o.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6veguus055kl8bfkeuo9.jpg) |
| __Webspace Information of Destination App Service Plan.__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e4xvjq0dxyncmf07s6l3.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vxr18j05vrymlrwpicxo.jpg) |

| STAGE #2 - PIPELINE TASK #1:- |
| --------- |
| Stage #2 will only execute if both the below conditions are met. |
| Condition 1: If the __Previous Stage__ executed successfully. |
| Condition 2: If the Source Branch from where the Pipeline is executed is the __"Main"__ Branch. |
| Once both the conditions are, Stage #2 will start executing but will wait for __Pipeline Approval__ first. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hvfduivsl739quboy0fl.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nx5x5u1xjpl8oc2hgl6y.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ezpkanzomfyl0uiqo788.jpg) |
| Post __Pipeline Approval__, Stage #2, Pipeline Task # will execute which is "__Settings Azure Account__" |

```
- stage: Migrate_WebApp_Between_AppServicePlans
  condition: |
     and(succeeded(), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')
     )
  jobs:
  - deployment: 
    displayName: Migrate WebApp Between App Service Plans
    environment: '$(envName)'
    pool:
      vmImage: $(BuildAgent)
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            displayName: Set Azure Account
            inputs:
              azureSubscription: $(ServiceConnection)
              scriptType: ps
              scriptLocation: inlineScript
              inlineScript: |
                az --version
                az account set --subscription ${{ parameters.SubscriptionID }}
                az account show
```

| STAGE #2 - PIPELINE TASK #2:- |
| --------- |
| Migrate Web App from Source to Destination App Service Plan. |
| The "__serverFarmId__" parameter in "__az webapp update__" command was found in the "__Web App Resource JSON__". |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/52rrzc2fzixpbxdtay8v.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2yltd9nht9utlcglggfk.jpg) |

```
- task: AzureCLI@2
            displayName: Migrate WebApp 
            inputs:
              azureSubscription: $(ServiceConnection)
              scriptType: ps
              scriptLocation: inlineScript
              inlineScript: |
                az webapp update -g ${{ parameters.RGName }} -n ${{ parameters.WebAppName }} --set serverFarmId=/subscriptions/${{ parameters.SubscriptionID }}/resourceGroups/${{ parameters.RGName }}/providers/Microsoft.Web/serverfarms/${{ parameters.AppServiceDestinationPlanName }}
                echo "###########################################################################################################################################################################################################"
                echo "WebApp ${{ parameters.WebAppName }} migrated from Source App Service Plan ${{ parameters.AppServiceSourcePlanName }} to Destination App Service Plan ${{ parameters.AppServiceDestinationPlanName }}!!!"
                echo "###########################################################################################################################################################################################################"
```

__NOW ITS TIME TO TEST!!!__

| TEST CASES:- |
| --------- |
| 1. Pipeline executed successfully. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zzmqes9jq5ja1eabpkvm.jpg) |
| 2. Web App is successfully migrated to Destination App Service Plan. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/twbrfnxer30uvzz8l39y.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gba4digrlm6q9l848mq1.jpg) |
| 3. Source App Service Plan is now empty. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k3errr458lrluqkadymh.jpg) |


__Hope You Enjoyed the Session!!!__

__Stay Safe | Keep Learning | Spread Knowledge__
