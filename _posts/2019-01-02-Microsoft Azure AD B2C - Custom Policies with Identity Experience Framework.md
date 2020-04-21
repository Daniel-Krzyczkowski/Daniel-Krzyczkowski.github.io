---
title: "Microsoft Azure AD B2C - Custom Policies with Identity Experience Framework"
excerpt: "In this article I would like to present how to use Identity Experience Framework together with custom policies which are designed primarily to address complex scenarios like connecting with external service during the registration."
header:
  image: /images/devisland/article15/assets/adbc1.png
---

<p align="center">
<img src="/images/devisland/article15/assets/adbc1.png?raw=true" alt="Microsoft Azure AD B2C - Custom Policies with Identity Experience Framework"/>
</p>

<h3><strong>Short introduction</strong></h3>
Azure Active Directory (Azure AD) is a multi-tenant, cloud-based directory and identity management service. It combines core directory services, application access management, and identity protection into a single solution. Now Azure Active Directory B2C (Business to Customers) is a separate service built on the same technology but not the same in functionality as Azure AD. The main difference is that Azure AD B2C  it is not to be used by single organization and its users. It allows any potential user to sign up with an email or social media provider like Facebook or Google. In this article I would like to present how to use Identity Experience Framework together with custom policies which are designed primarily to address complex scenarios like connecting with external service during the registration.

&nbsp;
<h3><strong>Structure and configuration</strong></h3>
There is great documentation on Microsoft Docs about how to setup Azure AD B2C together with Identity Experience Framework. You can find it <a href="https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started-custom" target="_blank" rel="noopener">here</a>. Once you add signing and encryption keys, register applications and download the starter pack with custom policies files, we can start from describing the structure of them.

Once you download custom policies files, please open "LocalAccounts" folder. You should see below files. It is very important to mention the structure of the custom policies and inheritance. Top level policy declaration is located in the "TrustFrameworkBase" file. Then "TrustFrameworkExtensions" file inherits from the previous file - it means that if you declare some new claim in the "TrustFrameworkBase" file it will be accessible in the "TrustFrameworkExtensions" file. To summarize each policy can have base policy declared:

```csharp
<BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
</BasePolicy>
```

Try to open each file and search for "BasePolicy". You will notice that each file has its base policy declared.

<strong>TrustFrameworkBase.xml</strong>

Contains most of the definitions. It is recommended that you make a minimum number of changes to this file to help with troubleshooting, and long-term maintenance of the policies. In this file you can define custom claims - for instance if you need to add some special information in the access token. Just right above of the "ClaimsSchema" tag you can add your additional claim definition:

```csharp
<ClaimType Id="userUniqueIdentifier">
        <DisplayName>userUniqueIdentifier</DisplayName>
        <DataType>string</DataType>
</ClaimType>
```

<strong>TrustFrameworkExtensions.xml</strong>

Holds the unique configuration changes for the tenant. This is the file where you can apply custom flow for the registration or login. Below is a fragment for the local account login from the original file:

```csharp
    <ClaimsProvider>
      <DisplayName>Local Account SignIn</DisplayName>
      <TechnicalProfiles>
         <TechnicalProfile Id="login-NonInteractive">
          <Metadata>
            <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
            <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
            <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
          </InputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
```

&nbsp;

<strong>SignUpOrSignIn.xml</strong>

Custom policy to hold the code responsible for user login or registration. Inherits from "TrustFrameworkExtensions.xml" file.

&nbsp;

<strong>PasswordReset.xml </strong>

Custom policy to hold the code responsible for password reset definition. Inherits from "TrustFrameworkExtensions.xml" file.

&nbsp;

<strong>ProfileEdit.xml</strong>

Custom policy to hold the code responsible for user profile edit. Inherits from "TrustFrameworkExtensions.xml" file.

&nbsp;
<h3><strong>Connecting external services</strong></h3>
I mentioned ath the beginning of this article that custom policies should be applied only to complex scenarios. For instance when during the registration you have to connect to the external service and validate user data (or insert user data to the external database). Microsoft recommends to use built-in policies. You can read about them more <a href="https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-reference-policies" target="_blank" rel="noopener">here</a>.

In this section I would like to present how to call external services during the registration and login process (in this case we will call Azure Function Apps).

<strong>Custom registration flow</strong>

Lets say that during the registration I would like to call Azure Function App which generates special code (GUID) and inserts this code together with user email in the Azure SQL database. In this case we have to declare new, custom claim provider. Open "TrustFrameworkExtensions.xml" file and search for "ClaimsProviders" section. Just right above closing tag of the "ClaimsProviders" add below code:

```csharp
  <ClaimsProvider>
      <DisplayName>Generate Special Code During User Reistration</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Users-Azure-Function-SignUp">
          <DisplayName>Get user email and send it to Azure Function</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/InsertNewUserFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="Users-Azure-Function-SignUp" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
```

Lets discuss the code a little bit with comments:

```csharp
 <!-- Custom claims provider definition-->
  <ClaimsProvider>
  <!-- Custom claims provider description-->
      <DisplayName>Generate Special Code During User Reistration</DisplayName>
      <TechnicalProfiles>
      <!-- Define technical profile for the custom claims provider-->
        <TechnicalProfile Id="Users-Azure-Function-SignUp">
          <DisplayName>Get user email and send it to Azure Function</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
           <!-- Here is the definition for the connection. As you can see we are connecting to the Azure Function App-->
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/InsertNewUserFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
           <!-- We want to pass a user email to the Azure Function App as a parameter-->
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
         <!-- We want to attach this flow to the existing technical profile connected with the registration which is declared in the TrustFrameworkBase file-->
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="Users-Azure-Function-SignUp" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
```

Now if you open "TrustFrameworkBase.xml" file you should be able to find technical profile called "LocalAccountSignUpWithLogonEmail". This is the original profile to which we are adding our custom flow with Azure Function App call.
Now once we have defined additional claims provider in the "LocalAccountSignUpWithLogonEmail" technical profile we should discover where this profile is used.

<strong>UserJourney</strong>

In the "TrustFrameworkBase.xml" file you can find "UserJourney" tag. This is the place where you can define custom the flow for the login or registration (you can define steps). If you search for the "SignUpOrSignIn" UserJourney you will find out that there are multiple orchestration steps inside this journey. OrchestrationStep with order Order="2" and Type="ClaimsExchange" contains "ClaimsExchange" tag with the technical profile we are looking for - "LocalAccountSignUpWithLogonEmail". Once user registers there is a call to the Azure Function described above.


<strong>Custom login flow</strong>

Above I described how to add custom step to the registration flow. Now during the login flow I would like to retrieve the special code generated for the user during the registration (generated by the Azure Function described above). This time we will add one more orchestration step to the user journey called "SignUpOrSignIn". Open "TrustFrameworkExtensions.xml" file and uncomment "UserJourneys" tags. Now copy the whole "SignUpOrSignIn" journey from the "TrustFrameworkBase.xml" file. Now "UserJourneys" section should look like below:

```csharp
  <UserJourneys>
	 <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
   
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
          <ClaimsProviderSelections>
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
            <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!-- This step reads any user attributes that we may not have received when in the token. -->
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>
 
        <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
	</UserJourneys
```

We need to add one more step during the login flow to call Azure Function App and retrieve the code for the user.  Search for “ClaimsProviders” section. Just right above "ClaimsProviders" tag add below code:

```csharp
    <ClaimsProvider>
      <DisplayName>Get User Special Code Azure Function</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="User-Code-Azure-Function-SignIn">
          <DisplayName>Get user special code</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/GetUserSpecialCodeFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="userCode" PartnerClaimType="userCode" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
```

This claims provider is declared to call Azure Function App and retrieve user special code basing on user email. Please note that we are also defining the technical profile here called "User-Code-Azure-Function-SignIn". We will use this techincal profile in the orchestration step which will be added to the "SignUpOrSignIn" user journey. Please look at the below code:

```csharp
	 <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
   
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
          <ClaimsProviderSelections>
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
            <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!-- This step reads any user attributes that we may not have received when in the token. -->
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="4" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="GetUserSpecialCode" TechnicalProfileReferenceId="User-Code-Azure-Function-SignIn" />
          </ClaimsExchanges>
        </OrchestrationStep>
 
        <OrchestrationStep Order="5" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
```

Please note that I added one more orchestration step (with order set to "4") and inside it I am using technical profile reference to the "User-Code-Azure-Function-SignIn". I also change the last step order to "5".

Whole "ClaimsProviders" section in the "TrustFrameworkExtensions.xml" file should look like below:

```csharp
  <ClaimsProviders>

    <ClaimsProvider>
      <DisplayName>Local Account SignIn</DisplayName>
      <TechnicalProfiles>
         <TechnicalProfile Id="login-NonInteractive">
          <Metadata>
            <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
            <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
            <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
          </InputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>

      <ClaimsProvider>
      <DisplayName>Generate Special Code During User Reistration</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Users-Azure-Function-SignUp">
          <DisplayName>Get user email and send it to Azure Function</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/InsertNewUserFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="Users-Azure-Function-SignUp" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>

       <ClaimsProvider>
      <DisplayName>Get User Special Code Azure Function</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="User-Code-Azure-Function-SignIn">
          <DisplayName>Get user special code</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/GetUserSpecialCodeFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="userCode" PartnerClaimType="userCode" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>

  </ClaimsProviders>
```

There is one more detail - very important to mention. As you can see in the above source code there is "PartnerClaimType" used in the "InputClaims" and "OutputClaims". This enables you to declare the name of the parameter which will be passed to the Azure Function App during the call and which will be returned from it. In this case we will pass "email" parameter to the Function App and it will return "userCode" parameter once successfully executed.

This is the content of the whole "TrustFrameworkExtensions.xml" file:

```csharp
<?xml version="1.0" encoding="utf-8" ?>
<TrustFrameworkPolicy 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" 
  PolicySchemaVersion="0.3.0.0" 
  TenantId="yourtenant.onmicrosoft.com" 
  PolicyId="B2C_1A_TrustFrameworkExtensions" 
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions">
  
  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase</PolicyId>
  </BasePolicy>
  <BuildingBlocks>

  </BuildingBlocks>

  <ClaimsProviders>

    <ClaimsProvider>
      <DisplayName>Local Account SignIn</DisplayName>
      <TechnicalProfiles>
         <TechnicalProfile Id="login-NonInteractive">
          <Metadata>
            <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
            <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
            <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
          </InputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>

      <ClaimsProvider>
      <DisplayName>Generate Special Code During User Reistration</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Users-Azure-Function-SignUp">
          <DisplayName>Get user email and send it to Azure Function</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/InsertNewUserFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="Users-Azure-Function-SignUp" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>

       <ClaimsProvider>
      <DisplayName>Get User Special Code Azure Function</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="User-Code-Azure-Function-SignIn">
          <DisplayName>Get user special code</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ServiceUrl">https://your-function-app.azurewebsites.net/api/GetUserSpecialCodeFunction?code=FunctionCode</Item>
            <Item Key="AuthenticationType">None</Item>
            <Item Key="SendClaimsIn">Body</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="objectId" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="userCode" PartnerClaimType="userCode" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>

  </ClaimsProviders>

  <UserJourneys>
	 <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
   
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
          <ClaimsProviderSelections>
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
            <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!-- This step reads any user attributes that we may not have received when in the token. -->
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="4" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="GetUserSpecialCode" TechnicalProfileReferenceId="User-Code-Azure-Function-SignIn" />
          </ClaimsExchanges>
        </OrchestrationStep>
 
        <OrchestrationStep Order="5" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
	</UserJourneys

</TrustFrameworkPolicy>
```


&nbsp;
<h3><strong>Test flow</strong></h3>

Once you finish editing policies files, upload them in the Azure portal. Remember to upload them in the correct order so start from "TrustFrameworkBase.xml" file and then upload "TrustFrameworkExtensions.xml" file and then rest of the files.

<img src="/images/devisland/article15/assets/AdBC2.png" alt="" width="775" height="432" class="size-full wp-image-1732 aligncenter" />

&nbsp;
<h3><strong>Wrapping up</strong></h3>

In this article I described (I hope so that at least it will help you discover how custom policies work) custom policies using Identity Experience Framework with Azure AD B2C. Custom policies are especially helpful when there is a custom flow required during the registration or login. Please remember that AD B2C provides built-in policies which can be used without writing and changing any code.