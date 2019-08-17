---
title: "Azure AD B2C Series - external service call during login and registration"
excerpt: "In this article would like to present how to call external service to get value of the custom claim in the Azure AD B2C Custom Policies (Identity Experience Framework)."
---

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall1.png?raw=true" alt="Azure AD B2C Series Custom Policies with custom claims"/>
</p>

I had a chance to work with the Azure Active Directory B2C quite a lot recently and decided that it would be nice to share some knowledge about it. Just to make life easier for people using it especially when there are some custom usage scenarios. This is the the second article from the series and in this article I would like to present how to call external service to get value of the custom claim in the Azure AD B2C Custom Policies (Identity Experience Framework).

There are also links to the great content after opening "Identity Experience Framework" tab in the Azure portal:

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall2.PNG?raw=true" alt="Image not found"/>
</p>

## Introduction

In the [previous article](https://daniel-krzyczkowski.github.io/Azure-AD-B2C-Series-Custom-Policies-With-Custom-Claims/) we added "extension_external_system_id" custom claim (attribute) to the final token. Value of this attribute was set to "external_system_id_1234" by default. This time we will fill the value of this claim using external service - in this case Azure Function. Let's see how to do it during the login and registration process.


## Create Azure Function instance

Before we move forward with custom policies cponfiguration, we have to create Azure Function instance. We will make a call to this Function during the registration and login process. I recommend to create Function App together with Application Insights connected because we will be able to see what parameters are passed in the calls.

Here is my Azure Function App configuration:

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall3.PNG?raw=true" alt="Image not found"/>
</p>

Now we have to create two HTTPTrigger functions with "POST" HTTP method enabled only.

**Create "GetExternalSystemIdOnRegistration" function**

First function will be responsible for filling "extension_external_system_id" claim during the registration process. Here is the source code:

```csharp
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("GetExternalSystemIdOnRegistration trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    log.LogInformation($"GetExternalSystemIdOnRegistration got request body: {data}");

    var email = data?.email;

    if(email == null)
    {
        new BadRequestObjectResult("Please pass user email in the request body");
    }

    Random rnd = new Random();
    int randomNumber  = rnd.Next(1, 100); 

    var external_system_id = string.Concat("NR_",email, "_", randomNumber);

    var externalSystemInfo = new ExternalSystemInfo
    {
       ExternalSystemId = external_system_id
    };

    var serializEdexternalSystemInfo = JsonConvert.SerializeObject(externalSystemInfo);

    log.LogInformation($"GetExternalSystemIdOnRegistration got external system id for user: {serializEdexternalSystemInfo}");

    return new OkObjectResult(externalSystemInfo);
}

class ExternalSystemInfo
{
    [JsonProperty("external_system_id")]
    public string ExternalSystemId {get; set;}
}
```

We have to pass user email in the request first. Then at the end random integer value is added to the email and assigned to the "external_system_id".

Please note that if you do not pass email properly, bad request result will be returned.

One more important note - the name of the returned parameter has to be exactly set to: "external_system_id". This is because in the AD B2C custom policy we will set it as a "PartnerClaimType" and policy will expect response parameter with such name.


## Call external service (Azure Function) during the registration process

In this scenario we would like to call Azure Function during the registration process and fill custom claim (attribute) called "extension_external_system_id".

**Download "TrustFrameworkExtensions.xml" policy file from the Azure portal:**

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall4.PNG?raw=true" alt="Image not found"/>
</p>

**Modify "TrustFrameworkExtensions.xml" policy file to make call to the Function App**

Open the file. In the "ClaimsProviders" block you will see that there is already "Local Account SignIn" Claims Provider declared and "Facebook" (if you use "SocialAndLocalAccounts" starter pack with custom policies files). Now we will add one more Claims Provider - Azure Function.

```csharp
     <ClaimsProvider>
      <DisplayName>Azure-Functions-Get-External-System-Id-On-Registration</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Azure-Functions-Get-External-System-Id-On-Registration">
          <DisplayName>Get external system ID for the user on registration</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">
            https://devislandb2c-functions.azurewebsites.net/api/GetExternalSystemIdOnRegistration?code=Bymakw6qap3ritZGd7/8bUD79og4z9Udqsl3xMxTnzWm3QWDJBIarw==
            </Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
          </InputClaims>
         <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="extension_external_system_id" PartnerClaimType="external_system_id" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="Azure-Functions-Get-External-System-Id-On-Registration" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
```

Let's discuss what is happening here.

First of all we have to declare Azure Function as a Claims Provider. In the "Metadata" block we have to provide Azure Function URL togethe with code parameter - it is required because if we do not add it, unauthorized response will be returned.

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall5.PNG?raw=true" alt="Image not found"/>
</p>

In the "InputClaim" we have to declare parameters which will be send to the function - in this case email of the user.

In the "OutputClaims" we have to declare parameters which will be returned by the Claims Provider - in this case "external_system_id".

**ClaimTypeReferenceId** - this is the name of the claim declared in the policy

**PartnerClaimType** - this is the name of the parameters passed to and from the external system

**SendClaimsIn** - means that we want to receive claims in the response body.

At the end we have to declare when such call to the function should be send. In this case we want to call Azure Function during the egistration step so we have to add it to the "LocalAccountSignUpWithLogonEmail" technical profile. This is done by adding "ValidationTechnicalProfile" reference.


**Test if final token contains external system ID for the user**

Now to check whether everything works as expected, run "signup_signin" policy, register new user and check if JWT token was returned.

Copy token, open [jwt.ms](https://jwt.ms) website and paste the token. Additional "extension_external_system_id" claim should be included:

<p align="center">
<img src="/images/devisland/article23/assets/B2cSeriesExternalServiceCall6.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
{
...
  "iat": 1565719626,
  "auth_time": 1565719626,
  "email": "danek22@op.pl",
  "name": "Daniel",
  "given_name": "Test",
  "family_name": "Test",
  "extension_external_system_id": "NR_danek22@op.pl_87",
  "tid": "19f4ca09-c2f9-4901-a62a-e47dbb3bc1e8"
}
```


## Call external service (Azure Function) during the login process

In this scenario we would like to call Azure Function during the login process and fill custom claim (attribute) called "extension_external_system_id".

**Create "GetExternalSystemIdOnLogin" function**

Second function will be responsible for filling "extension_external_system_id" claim during the login process. Here is the source code (it is quite the same like above):

```csharp
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
 log.LogInformation("GetExternalSystemIdOnLogin trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    log.LogInformation($"GetExternalSystemIdOnLogin got request body: {data}");

    var email = data?.email;

    if(email == null)
    {
        new BadRequestObjectResult("Please pass user email in the request body");
    }

    Random rnd = new Random();
    int randomNumber  = rnd.Next(1, 100); 

    var external_system_id = string.Concat("NR_",email, "_", randomNumber);

    var externalSystemInfo = new ExternalSystemInfo
    {
       ExternalSystemId = external_system_id
    };

    var serializEdexternalSystemInfo = JsonConvert.SerializeObject(externalSystemInfo);

    log.LogInformation($"GetExternalSystemIdOnLogin got external system id for user: {serializEdexternalSystemInfo}");

    return new OkObjectResult(externalSystemInfo);
}

class ExternalSystemInfo
{
    [JsonProperty("external_system_id")]
    public string ExternalSystemId {get; set;}
}
```

**Modify "TrustFrameworkExtensions.xml" policy file to make call to the Function App**

Open the file. In the "ClaimsProviders" block add below Claims Provider - just under the previous one we created for the registration flow:

```csharp
    <ClaimsProvider>
      <DisplayName>Azure-Functions-Get-External-System-Id-On-Login</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Azure-Functions-Get-External-System-Id-On-Login">
          <DisplayName>Get external system ID for the user on login</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">
            https://devislandb2c-functions.azurewebsites.net/api/GetExternalSystemIdOnLogin?code=fdHN49JsRof2jyksv2u7ACMBEyDDKR4Fy8RUIgjiajIemEKhCtUJIQ==
            </Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="signInNames.emailAddress" PartnerClaimType="email" />
          </InputClaims>
         <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="extension_external_system_id" PartnerClaimType="external_system_id" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
```

Please note that this time we changed "ClaimTypeReferenceId" from the input claim. This is because email address during the login flow is available under "signInNames.emailAddress" id.

URL for the Function was taken from the Azure portal. Note that code parameter is added too.


**Move User Journey with id "SignUpOrSignIn" to the "TrustFrameworkExtensions.xml" policy file from the "TrustFrameworkBase.xml" policy file**

To enable call to the Azure Function during the login flow we have to add one more step to the "SignUpOrSignIn" User Journey. If you are not familiar with the User Journeys I encourage you to check [documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/userjourneys).

Cut whole User Journey with id "SignUpOrSignIn" from the "TrustFrameworkBase.xml" file:

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
        <!-- Check if the user has selected to sign in using one of the social providers -->
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
        <!-- For social IDP authentication, attempt to find the user account in the directory. -->
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
        <!-- Show self-asserted page only if the directory does not have the user account already (i.e. we do not have an objectId). 
          This can only happen when authentication happened using a social IDP. If local account was created or authentication done
          using ESTS in step 2, then an user account must exist in the directory by this time. -->
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
        <!-- This step reads any user attributes that we may not have received when authenticating using ESTS so they can be sent 
          in the token. -->
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
        <!-- The previous step (SelfAsserted-Social) could have been skipped if there were no attributes to collect 
             from the user. So, in that case, create the user in the directory if one does not already exist 
             (verified using objectId which would be set from the last step if account was created in the directory. -->
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
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
```

Paste it in the "TrustFrameworkExtensions.xml" file between "User Journeys" block (uncomment it first):

```csharp
  <UserJourneys>
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
        <!-- Check if the user has selected to sign in using one of the social providers -->
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
        <!-- For social IDP authentication, attempt to find the user account in the directory. -->
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
        <!-- Show self-asserted page only if the directory does not have the user account already (i.e. we do not have an objectId). 
          This can only happen when authentication happened using a social IDP. If local account was created or authentication done
          using ESTS in step 2, then an user account must exist in the directory by this time. -->
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
        <!-- This step reads any user attributes that we may not have received when authenticating using ESTS so they can be sent 
          in the token. -->
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
        <!-- The previous step (SelfAsserted-Social) could have been skipped if there were no attributes to collect 
             from the user. So, in that case, create the user in the directory if one does not already exist 
             (verified using objectId which would be set from the last step if account was created in the directory. -->
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
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
	</UserJourneys>
```

Now as you can see there are 7 "Orchestration Steps". Those steps are executed during the login process to construct the final token.

Add below step with "Order" set to "7" and set "Order" for the "SendClaims" step to "8":

```csharp
 <OrchestrationStep Order="7" Type="ClaimsExchange">
     <ClaimsExchanges>
       <ClaimsExchange Id="GetExternalSystemIdForTheUser" TechnicalProfileReferenceId="Azure-Functions-Get-External-System-Id-On-Login" />
        </ClaimsExchanges>
   </OrchestrationStep>
 <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
```

Final "SignUpOrSignIn" User Journey should look like below:

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
        <!-- Check if the user has selected to sign in using one of the social providers -->
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
        <!-- For social IDP authentication, attempt to find the user account in the directory. -->
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
        <!-- Show self-asserted page only if the directory does not have the user account already (i.e. we do not have an objectId). 
          This can only happen when authentication happened using a social IDP. If local account was created or authentication done
          using ESTS in step 2, then an user account must exist in the directory by this time. -->
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
        <!-- This step reads any user attributes that we may not have received when authenticating using ESTS so they can be sent 
          in the token. -->
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
        <!-- The previous step (SelfAsserted-Social) could have been skipped if there were no attributes to collect 
             from the user. So, in that case, create the user in the directory if one does not already exist 
             (verified using objectId which would be set from the last step if account was created in the directory. -->
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
            <ClaimsExchange Id="GetExternalSystemIdForTheUser" TechnicalProfileReferenceId="Azure-Functions-Get-External-System-Id-On-Login" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
```

**Test if final token contains external system ID for the user**

Now to check whether everything works as expected, run "signup_signin" policy, try to login with the previously created user and check if JWT token was returned.

Copy token, open [jwt.ms](https://jwt.ms) website and paste the token. Additional "extension_external_system_id" claim should be included and its value should be different than value returned during the registration (because it is generated randomly):

```csharp
{
...
  "iat": 1565720125,
  "auth_time": 1565720125,
  "name": "Daniel",
  "given_name": "Test",
  "family_name": "Test",
  "extension_external_system_id": "NR_danek22@op.pl_96",
  "tid": "19f4ca09-c2f9-4901-a62a-e47dbb3bc1e8"
}
```


# Summary

In this article I presented how to call external service (in this case Azure Function) to get value of the custom claim in the Azure AD B2C Custom Policies (Identity Experience Framework). There are a lot of custom scenarios where call to external service is required so I hope you found this article interesting. In the next article I will present how to log errors in the custom policies usign Azure Application Insights.