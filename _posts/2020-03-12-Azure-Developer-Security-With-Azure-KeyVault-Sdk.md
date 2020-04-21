---
title: "Secure Application Development With Azure Key Vault SDK and Secret Manager"
excerpt: "In this article, I would like to present how to integrate Azure Key Vault .NET SDK to eliminate storing credentials in the source code"
header:
  image: /images/devisland/article33/assets/SecureApplicationDevelopment1.png
---

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment1.png?raw=true" alt="Secure Application Development With Azure Key Vault SDK and Secret Manager"/>
</p>

## Introduction

Almost every application uses some credentials. It can be a database's connection string or storage's connection string. The biggest challenge for local development is how to eliminate storing credentials and secrets directly in the source code. This is why I would like to present how to use Secret Manager tool together with Azure Key Vault .NET SDK and Azure Identity .NET SDK to access secrets stored in the Azure Key Vault.

## Challenge

The main challenge is to eliminate storing any credentials/secrets in the source code and avoid pushing them to the source code repository.

## Solution

In this example, we will use ASP .NET Core Web API application template. You can find the sample on my GitHub [here](https://github.com/Daniel-Krzyczkowski/AzureDeveloperTemplates/tree/master/src/azure-key-vault-sdk-asp-net-core-template).

We will use two Azure SDK libraries there:

1. [Azure.Security.KeyVault.Secrets](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)
2. [Azure.Identity](https://www.nuget.org/packages/Azure.Identity/)


<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment2.PNG?raw=true" alt="Image not found"/>
</p>

In the "Infrastructure" project, there is "ISecretManager" interface:

```csharp
    public interface ISecretManager
    {
        Task<string> GetSecretAsync(string secretName);
        Task SetSecretAsync(string secretName, string secretValue);
        Task DeleteSecret(string secretName);
        Task UpdateSecret(string secretName, string secretValue);
    }
```

As you we see there are methods to retrieve secrets from the secret store, adding secrets, deleting and updating. In the implementation of this interface we can find "_secretClient" variable that is the instance of the "SecretClient" class provided by the Azure SDK library - "Azure.Security.KeyVault.Secrets":

```csharp
    public class SecretManager : ISecretManager
    {
        protected readonly IKeyVaultSecretClientClientFactory _keyVaultSecretClientClientFactory;
        private readonly SecretClient _secretClient;

        public SecretManager(IKeyVaultSecretClientClientFactory keyVaultSecretClientClientFactory)
        {
            _keyVaultSecretClientClientFactory = keyVaultSecretClientClientFactory;
            _secretClient = _keyVaultSecretClientClientFactory.SecretClient;
        }

        public async Task<string> GetSecretAsync(string secretName)
        {
            KeyVaultSecret secret = await _secretClient.GetSecretAsync(secretName);
            if (secret != null)
            {
                return secret.Value;
            }

            else
            {
                return string.Empty;
            }
        }

        public async Task SetSecretAsync(string secretName, string secretValue)
        {
            await _secretClient.SetSecretAsync(secretName, secretValue);
        }

        public async Task DeleteSecret(string secretName)
        {
            DeleteSecretOperation operation = await _secretClient.StartDeleteSecretAsync(secretName);
        }

        public async Task UpdateSecret(string secretName, string secretValue)
        {
            await SetSecretAsync(secretName, secretValue);
        }
    }
```

"IKeyVaultSecretClientClientFactory" interface and its implementation were created to inject "SecretClient" and making sure that it is created as a singleton:

```csharp
    public class KeyVaultSecretClientClientFactory : IKeyVaultSecretClientClientFactory
    {
        public KeyVaultSecretClientClientFactory(SecretClient secretClient)
        {
            SecretClient = secretClient;
        }

        public SecretClient SecretClient { get; }
    }
```

Once above classes are implemented we can register their instances. This is done in the "ServiceCollectionExtensions" static class:

```csharp
    public static class ServiceCollectionExtensions
    {
        public static void RegisterDependencies(this IServiceCollection services, IConfiguration configuration)
        {
            var keyVaultSecretClientClientFactory = InitializeSecretClientInstanceAsync(configuration);
            services.AddSingleton<IKeyVaultSecretClientClientFactory>(keyVaultSecretClientClientFactory);
            services.AddSingleton<ISecretManager, SecretManager>();
        }

        private static KeyVaultSecretClientClientFactory InitializeSecretClientInstanceAsync(IConfiguration configuration)
        {
            string keyVaultUrl = configuration["KeyVaultSettings:Url"];

            TokenCredential credential = new DefaultAzureCredential();
#if DEBUG
            credential = new ClientSecretCredential(configuration["AZURE_TENANT_ID"],
                                                    configuration["AZURE_CLIENT_ID"],
                                                    configuration["AZURE_CLIENT_SECRET"]);
#endif

            var secretClient = new SecretClient(new Uri(keyVaultUrl), credential);
            var keyVaultSecretClientClientFactory = new KeyVaultSecretClientClientFactory(secretClient);
            return keyVaultSecretClientClientFactory;
        }
    }
```

Please note that we are using only "keyVaultUrl" parameter without any client id and client secret values stored in the source code. What is more, we do not load any secrets from the "app.settings.json" file - only the Key Vault Url:

```csharp
  "KeyVaultSettings": {
    "Url": ""
  }
```

##### Great! But how the Key Vault secrets are accessed then?

In this case all the magic is done by the "Azure.Identity" library. Please note that we use "DefaultAzureCredential" when creating "SecretClient" instance. There is great explanation of how exactly this mechanism works on the [Azure.Identity GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity#authenticating-with-defaultazurecredential).

There is an important fragment from the documentation:

*The DefaultAzureCredential will first attempt to authenticate using credentials provided in the environment. In a development environment you can authenticate as a service principal with the DefaultAzureCredential by providing configuration in environment variables as described in the next section.*

*When executing this in a development machine you need to first configure the environment setting the variables AZURE_CLIENT_ID, AZURE_TENANT_ID and AZURE_CLIENT_SECRET to the appropriate values for your service principal.*

To create new Service Principal, we can use this command in the Azure CLI:

```csharp
az ad sp create-for-rbac -n <your-application-name> --skip-assignment
```

Please note that first we will have to sign in to the Azure using below command:

```csharp
az login --tenant <name-of-your-azure-tenant>
```

Once command is applied, there should be a response like below:

```csharp
{
  "appId": "dc314037-c67f-490b-a3b3-7fb42c2d53a8",
  "displayName": "sample-web-api",
  "name": "http://sample-web-api",
  "password": "62825e3d-0343-4ed1-9c70-a9665063aca6",
  "tenant": "xxx"
}
```

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment3.PNG?raw=true" alt="Image not found"/>
</p>

Great - now we have Service Principal registered in the Azure Active Directory. We can also check it in the Azure portal, in the Azure Active Directory tab under "App registrations":

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment5.PNG?raw=true" alt="Image not found"/>
</p>

Next step is to enable access for it in the Azure Key Vault. To do it we have to open Key Vault blade in the Azure portal and select "Access policies":

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment5.PNG?raw=true" alt="Image not found"/>
</p>

There we have to click "+Add Access Policy" button. Then we have to click the "Select principal" section and search for the App ID that was returned above. In my case it was: "dc314037-c67f-490b-a3b3-7fb42c2d53a8". Once selected we have to click "Select" button:

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment6.PNG?raw=true" alt="Image not found"/>
</p>

In the "Secret permissions" we have to select "Get and List":

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment7.PNG?raw=true" alt="Image not found"/>
</p>

Then we can click "Add" button and next click "Save" button to save all changes. Now our Service Principal (application) has access to the secrets stored in the Key Vault. Now it is time to store AZURE_CLIENT_ID, AZURE_TENANT_ID and AZURE_CLIENT_SECRET variables in the Secret Manager.

##### Use Secret Manager to store variables

[The Secret Manager](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-3.1&tabs=windows) tool stores sensitive data during the development of an ASP.NET Core project. It is very helpful because we can avoid storing any sensitive data directly in the source code.

Important note from the documentation:

*The Secret Manager tool doesn't encrypt the stored secrets and shouldn't be treated as a trusted store. It's for development purposes only. The keys and values are stored in a JSON configuration file in the user profile directory.*

There are few steps we have to do. First, we have to enable secret storage. To do it, we have to open command line an move to the location where our Web API project is located, in my case:

```csharp
cd C:\Users\DanielKrzyczkowski\Desktop\azure-key-vault-sdk-asp-net-core-template\AzureDeveloperTemplates.KeyVaultSdk.WebAPI
```

Note that the preceding command adds a "UserSecretsId" element within a PropertyGroup of the .csproj file:

```csharp
  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <UserSecretsId>8b540751-4814-42c0-b8e5-188ca828a49d</UserSecretsId>
  </PropertyGroup>
```

Now we are ready to store AZURE_CLIENT_ID, AZURE_TENANT_ID and AZURE_CLIENT_SECRET variables generated earlier using below commands:

```csharp
dotnet user-secrets set "AZURE_CLIENT_ID" "dc314037-c67f-490b-a3b3-7fb42c2d53a8"

dotnet user-secrets set "AZURE_TENANT_ID" "xxx"

dotnet user-secrets set "AZURE_CLIENT_SECRET" "62825e3d-0343-4ed1-9c70-a9665063aca6"
```

There should be information displayed:

*Successfully saved AZURE_CLIENT_SECRET = 62825e3d-0343-4ed1-9c70-a9665063aca6 to the secret store.*

Great, once we have these calues in the secret store, we can avoid storing them directly in the source code of our application.

##### Access Key Vault secrets from the local machine

We can open now the Key Vault blade in the Azure portal and add one secret - for instance "TestSecret":

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment8.PNG?raw=true" alt="Image not found"/>
</p>

Now let's get back to the application source code. We can now use instance of the "SecretManager" class to access our secret from the Key Vault. Just for a test we can do it in the "ServiceCollectionExtensions" class inside "RegisterDependencies" method:

```csharp
var testSecret = await _secretManager.GetSecretAsync("TestSecret");
```
We have to also remember to add URL in the "app.settings.json" file:

```csharp
  "KeyVaultSettings": {
    "Url": "https://kv-azure-dev.vault.azure.net/"
  }
```

Let's start the application and see if secret is successfully retrieved:

<p align="center">
<img src="/images/devisland/article33/assets/SecureApplicationDevelopment9.png?raw=true" alt="Image not found"/>
</p>


## Summary

In this article I explained how to use the Secret Manager tool together with Azure Key Vault .NET SDK and Azure Identity .NET SDK to access secrets stored in the Azure Key Vault. In this way developers can avoid storing credentials and secrets directly in the source code and of course at the end, source code pushed to the repository does not contain them.
