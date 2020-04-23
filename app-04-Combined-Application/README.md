# Lab 01 - Cognitive Services
# Cognitive Services and real-time Serverless with Signal R

Let's start with a workshop overview. 
The main idea is to build a real-time serverless application with Translation and "Speech to text" Cognitive services from Microsoft.
Your goal is to use this tutorial to build your idea. The tutorial provides links to Cognitive services documentation, scenarios, etc.

Requirements for this tutorial are Visual Studio 2019 community or VS Code. Basic knowledge of C#

Below are three steps to build a SignalR real-time app, connect it to translation services, and then use Speech to Text.

# Step 1. Infrastructure

1. The first step is to create a demo Azure subscription 
https://azure.microsoft.com/en-us/free/
Take script below to deploy all needed infrastructure, be aware that cognitive services deployed with production tier S0, you can change S0 to F0 to use the free tier. The main limitation of F0 is the single active concurrent request.

```
    subscriptionID=$(az account list --query "[?contains(name,'Microsoft')].[id]" -o tsv)
    echo "Test subscription ID is = " $subscriptionID
    az account set --subscription $subscriptionID
    az account show

    location=northeurope
    postfix=$RANDOM
    groupName=AIServerlessUA$postfix

    az group create --name $groupName --location $location

    location=northeurope
    accountSku=Standard_LRS
    accountName=${groupName,,}
    echo "accountName  = " $accountName

    az storage account create --name $accountName --location $location --kind StorageV2 \
    --resource-group $groupName --sku $accountSku --access-tier Hot  --https-only true

    runtime=dotnet
    applicationName=${groupName,,}
    accountName=${groupName,,}
    echo "applicationName  = " $applicationName

    az functionapp create --resource-group $groupName \
    --name $applicationName --storage-account $accountName --runtime $runtime \
    --consumption-plan-location $location --functions-version 3

    signalName=${groupName,,}
    az signalr create --name $signalName --resource-group $groupName --sku Standard_S1 --unit-count 1 --service-mode Serverless

    signalConnString=$(az signalr key list --name $(az signalr list \
    --resource-group $groupName --query [0].name -o tsv) \
    --resource-group $groupName --query primaryConnectionString -o tsv)

	#add cors if you know your domains already.
	#az signalr cors add --name $signalName --resource-group $groupName --allowed-origins "http://example1.com" "https://example2.com"
	signalKey=$(az signalr key list --name $signalName --resource-group $groupName --query primaryKey -o tsv)

	speechName=Speech${groupName,,}
	textName=Text${groupName,,}
	az cognitiveservices account list-skus --location $location --kind SpeechServices

	az cognitiveservices account create --name $speechName --resource-group $groupName --location $location \
	--kind SpeechServices --sku S0 --yes

	speechKey1=$(az cognitiveservices account keys list --name $speechName --resource-group $groupName --query "key1" -o tsv)

	az cognitiveservices account create --name $textName --resource-group $groupName --location $location \
	--kind TextTranslation --sku S0 --yes

	textKey1=$(az cognitiveservices account keys list --name $textName --resource-group $groupName --query "key1" -o tsv)

	printf "\n\n Change <speechKey1> with:\n$speechKey1\n\n"

	printf "\n\n Change <textKey1> with:\n$textKey1\n\n"

	printf "\n Replace <signalKey> with:\n$signalKey\n\n"

	printf "\n\nReplace <signalConnString> with:\n$signalConnString\n\n"
```

FYI, the link below provides a reference list of Cognitive Services types with CLI reference
https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account-cli?tabs=windows

3. Copy SignalR connection string and Cognitive service keys to notepad for the further usage.
