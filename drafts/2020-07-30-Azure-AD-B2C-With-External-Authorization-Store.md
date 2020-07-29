---
title: "Azure AD B2C with external authorization store"
excerpt: "This article presents how to implement external authorization store with Azure AD B2C identity service"
header:
  image: /images/devisland/article39/assets/AzureAdB2cExternalAuthorization1.png
---

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization1.png?raw=true" alt="Azure AD B2C with external authorization store"/>
</p>

# Introduction

I had a chance to work with the Azure Active Directory B2C quite a lot recently and decided that it would be nice to share some knowledge about it. Just to make life easier for people using it especially when there are some custom usage scenarios. This is another article related to the Azure AD B2C identity service. Before we start with technical details I would like to quickly talk about authorization. Probably you know it but just to remind:

1. Authentication - the act of validating that users are who they claim to be. You can try to imagine a guest in the hotel at the reception. We have to provide a passport or other ID to prove that we are who we are.
2. Authorization -  The process of giving the user permission to access a specific resource or function. Regarding the above example with the hotel. We will get the key that can be used to open only one specific room. We will not have the key to open all the rooms at the hotel.

When talking about Azure AD B2C identity service it is worth knowing that it is service that can be used to authenticate users. There is no authorization mechanism/layer in the AD B2C (yet). Of course, this is not the issue because we can leverage other great Azure services and build our authorization store to manage user's permissions. In this article, I would like to present how to use Azure AD B2C together with Azure Functions and Azure SQL database to create an authorization store for users.

# Identity Experience Framework (custom policies) required

To be able to build an authorization store and connect it with Azure AD B2C, we have to use Identity Experience Framework (custom policies). With custom policies, we can extend functionality that AD B2C provides. We can for instance call external service during the user's login or registration. If you would like to read more about Identity Experience Framework, please read [this](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started?tabs=applications) documentation.


## Solution architecture

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization2.png?raw=true" alt="Image not found"/>
</p>

The below diagram presents a solution where we use a custom authorization store connected with Azure AD B2C. As we can see there are Azure Functions used which are great for this kind of scenario. In this solution there are three Azure Functions:

1. get-user-authorization-groups-identifiers-func - get IDs of groups to which user belongs
2. get-user-authorization-groups-func - get IDs of groups together with names to which user belongs
3. get-authorization-groups-func - get all authorization groups to which user can be assigned

First function is called by Azure AD B2C during user login flow. Function calls Azure SQL database then and gets all group identifiers to which user belongs. These identifiers are returned to the AD B2C and stored in the JWT token returned to web application.

Second function app returns all user's groups together with names and the third function app provides information about all possible authorization groups. The third function is called by the web application on startup.

**IMPORTANT!**

In this scenario, we embed groups in the user's JWT token. If there will be a lot of groups the good idea is to move group verification to web application logic. This will help avoid storing groups in the JWT token.

Of course, my solution is just the proof of concept but in the real world, we would store all authorization groups in one database table. In another table, we could store permissions for these groups and in the third table we could store assigned users to specific groups.

## Custom policy modification

To be able to call Azure Function and embed information about authorization groups in the JWT token we have to declare custom claim and modify policies.

### Trust Framework Base policy

In the *TrustFrameworkBase* policy we have to add custom claim:

```csharp
    <ClaimType Id="extension_authorization_groups">
      <DisplayName>User authorization groups</DisplayName>
      <DataType>stringCollection</DataType>
    <UserHelpText>Authorization groups to which user belongs.</UserHelpText>
    </ClaimType>
```

As we can see this is a collection of string values, in our case we will store user's authorization groups there.

### Trust Framework Extensions policy

To be able to call Azure Functions on user's login, we have to add custom claims provider in the *TrustFrameworkExtensions* policy in the *ClaimsProviders* section:

```csharp
    <ClaimsProvider>
      <DisplayName>Get-User-Authorization-Groups-On-Login</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Get-User-Authorization-Groups-On-Login">
          <DisplayName>Get authorization groups from external authorization store for the user on login</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://func-clean-arch-authorization.azurewebsites.net/api/get-user-authorization-groups-identifiers-func?code=xxx</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="AllowInsecureAuthInProduction">true</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="userId" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="extension_authorization_groups" PartnerClaimType="authorizationGroups" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
```

Please note that we are passing *objectId* as *userId* to function request as a parameter. We expect to receive a response that will contain *authorizationGroups* payload that will be stored in the *extension_authorization_groups* claim in the JWT token.

We have to also move *SignUpOrSignIn* user journey from *TrustFrameworkBase* to *TrustFrameworkExtensions* policy. As we can see step 7 below is responsible for calling Azure Functions using claims provider that we declared above.

```csharp
      <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
      
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
          <ClaimsProviderSelections>
            <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange" />
            <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmailExchange" />
          </ClaimsProviderSelections>
          <ClaimsExchanges>
            <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
              <Value>objectId</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
            <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
              <Value>authenticationSource</Value>
              <Value>localAccountAuthentication</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadUsingAlternativeSecurityId" TechnicalProfileReferenceId="AAD-UserReadUsingAlternativeSecurityId-NoError" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="4" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
              <Value>objectId</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="SelfAsserted-Social" TechnicalProfileReferenceId="SelfAsserted-Social" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="5" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
              <Value>authenticationSource</Value>
              <Value>socialIdpAuthentication</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="6" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
              <Value>objectId</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserWrite" TechnicalProfileReferenceId="AAD-UserWriteUsingAlternativeSecurityId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="7" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="Get-User-Authorization-Groups-On-Login" TechnicalProfileReferenceId="Get-User-Authorization-Groups-On-Login" />
          </ClaimsExchanges>
        </OrchestrationStep>
 
        <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
 
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
```

### Sign up or sign in policy

The last step is to declare *extension_authorization_groups* claim as *OutputClaim* in the *SignUpOrSignin* policy. This will add authorizations groups to the JWT token returned to the web application.

```csharp
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
        <OutputClaim ClaimTypeReferenceId="extension_account_type" />
        <OutputClaim ClaimTypeReferenceId="extension_authorization_groups" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
```

Once user signs in, there is information about authorization groups included in the JWT token:

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization6.PNG?raw=true" alt="Image not found"/>
</p>


## Web application configuration

The last step is to secure web application with Azure AD B2C and call Azure Functions to get all authorization groups at application startup.

### Load authorization groups at application startup

In this scenario, I used ASP .NET Core Razor Pages web application. I wrote an extension method called *AddAuthorizationServices* that is invoked in the *Startup* class when the application is launched. In this extension method, we call Azure Function app to get list of all authorization groups. Then we call *services.AddAuthorization* to enable policy-based authorization for the web application.


```csharp
        public static IServiceCollection AddAuthorizationServices(this IServiceCollection services)
        {
            services.AddHttpClient<IAuthorizationGroupService, AuthorizationGroupService>();
            var serviceProvider = services.BuildServiceProvider();

            var authorizationService = serviceProvider.GetRequiredService<IAuthorizationGroupService>();

            var authorizationGroups = authorizationService.GetAuthorizationGroups().GetAwaiter().GetResult();

            services.AddAuthorization(options =>
            {
                foreach (var authorizationGroup in authorizationGroups)
                {
                    options.AddPolicy(
                        authorizationGroup.GroupName,
                        policy =>
                            policy.AddRequirements(new MemberOfGroupRequirement(authorizationGroup.GroupName, authorizationGroup.Id.ToString())));
                }
            });

            services.AddSingleton<IAuthorizationHandler, MemberOfGroupHandler>();

            return services;
        }
```

The code above will configure authorization basing on the groups that were provided by the Azure Function app, retrieved from the Azure SQL database.


### Apply authorization on the specific view

To verify whether authorization works properly, in the *PrivateContentModel* class I added *[Authorize(Policy = "Employee")]* attribute. This will make sure that only user who belongs to the *Employee* authorization group, will have access to the private content:

```csharp
    [Authorize(Policy = "Employee")]
    [Authorize]
    public class PrivateContentModel : PageModel
```

When we launch the application we can see the result:

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization3.PNG?raw=true" alt="Image not found"/>
</p>


## Azure SQL Database tables

As mentioned above information about authorization groups and user's assignments is stored in the Azure SQL database. Here is the structure of these two tables:

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article39/assets/AzureAdB2cExternalAuthorization4.PNG?raw=true" alt="Image not found"/>
</p>


## Azure Function Apps

In the Azure Functions project I used [Dapper](https://github.com/StackExchange/Dapper) to prepare database queries:

```csharp
        public async Task<IEnumerable<UserAuthorizationGroup>> GetAuthorizationGroupsForUserAsync(Guid userId)
        {
            using (var connection = new SqlConnection(_azureSqlDatabaseConfiguration.ConnectionString))
            {
                connection.Open();

                var authorizationGroups = await connection.QueryAsync<UserAuthorizationGroup>(
                   @"select Id, UserId, GroupId FROM dbo.UserAuthorizationGroups
                     WHERE UserId=@userID", new { userId });

                if (authorizationGroups.AsList().Count == 0)
                {
                    throw new KeyNotFoundException();
                }

                else
                {
                    return authorizationGroups;
                }
            }
        }

        public async Task<IEnumerable<AuthorizationGroup>> GetAuthorizationGroupsAsync()
        {
            using (var connection = new SqlConnection(_azureSqlDatabaseConfiguration.ConnectionString))
            {
                connection.Open();

                var authorizationGroups = await connection.QueryAsync<AuthorizationGroup>(
                   @"select Id, GroupName FROM dbo.AuthorizationGroups");

                if (authorizationGroups.AsList().Count == 0)
                {
                    throw new KeyNotFoundException();
                }

                else
                {
                    return authorizationGroups;
                }
            }
        }
```

I encourage you to review the source code. The link is mentioned in the summary of this article. Here is the source code of all three function apps:

```csharp
    public class AuthorizationHandlerFunc
    {
        private readonly IAuthorizationQueries _authorizationQueries;

        public AuthorizationHandlerFunc(IAuthorizationQueries authorizationQueries)
        {
            _authorizationQueries = authorizationQueries;
        }

        [FunctionName("get-user-authorization-groups-identifiers-func")]
        public async Task<IActionResult> GetUserAuthorizationGroupsIdentifiersAsync(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            return await GetUserGroupsDataAsync(req, false, log);
        }

        [FunctionName("get-user-authorization-groups-func")]
        public async Task<IActionResult> GetUserAuthorizationGroupsAsync(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            return await GetUserGroupsDataAsync(req, true, log);
        }

        [FunctionName("get-authorization-groups-func")]
        public async Task<IActionResult> GetAuthorizationGroupsAsync(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation($"{nameof(GetAuthorizationGroupsAsync)} HTTP trigger function received a request");

            var authorizationGroups = await _authorizationQueries
                                                .GetAuthorizationGroupsAsync();

            log.LogInformation("Successfully retrieved authorization groups");

            return new OkObjectResult(authorizationGroups);
        }

        private async Task<IActionResult> GetUserGroupsDataAsync(HttpRequest req, bool includeGroupNames, ILogger log)
        {
            log.LogInformation($"{nameof(GetUserAuthorizationGroupsAsync)} HTTP trigger function received a request");


            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            var userId = data.userId;
            string userIdAsString = userId.ToString();

            if (userId == null)
            {
                log.LogInformation("UserId parameter is required");
                return new BadRequestResult();
            }

            else
            {
                var userAuthorizationGroups = await _authorizationQueries
                                                    .GetAuthorizationGroupsForUserAsync(Guid.Parse(userIdAsString));

                log.LogInformation("Successfully retrieved authorization groups for specific user");

                if (includeGroupNames == true)
                {
                    return new OkObjectResult(new UserAuthorizationGroupsDto
                    {
                        AuthorizationGroups = userAuthorizationGroups
                    });
                }

                else
                {
                    return new OkObjectResult(new UserAuthorizationGroupsIdentifiersDto
                    {
                        AuthorizationGroups = userAuthorizationGroups.Select(x => x.GroupId)
                    });
                }
            }
        }
    }
```


# Summary

In this article, we talked about adding custom authorization to the Azure AD B2C identity service. With Identity Experience Framework it is possible to extend default functionalities of Azure AD B2C. Source code of this solution is available on my Github [here](https://github.com/Daniel-Krzyczkowski/IdentityDeveloperTemplates). This solution can be used to manage permissions for different kinds of users.
