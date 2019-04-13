---
title: "Easy mocking in C# code with FakeItEasy library"
---

<p align="center">
<img src="/images/devisland/article6/assets/fakeit11.png?raw=true" alt="Easy mocking in C# code with FakeItEasy library"/>
</p>

<h3><strong>Short introduction</strong></h3>
Web API access can be protected to avoid unauthorized access. In this article I would like to present how to configure Azure Active Directory B2C (Business-to-Consumer). Before that its worth to mention few words about Azure AD  (Azure AD).

Azure Active Directory is a cloud identity provider service or Identity as a Service (IdaaS) provided by Microsoft.

Azure AD B2C is a separate service (with same technology as standard Azure AD) which allows organizations to build a cloud identity directory for their customers.

&nbsp;
<h3><strong>Setup Azure Active Directory B2C</strong></h3>
Once you sign in to <a href="https://portal.azure.com" target="_blank" rel="noopener">Microsoft Azure Portal</a> (Azure subscription is required here) click "Create resource" in the left top corner:
<p style="text-align:center;"><img class="alignnone wp-image-234" src="/images/devisland/article6/assets/adb2c2.png?w=300" alt="" width="307" height="178" /></p>
In search window type "azure b2c" and select "Azure Active Directory B2C" resource. Click "Create" button:
<p style="text-align:center;"><img class="alignnone wp-image-236" src="/images/devisland/article6/assets/adb2c3.png?w=300" alt="" width="477" height="116" /></p>
In the next tab select "Create a new Azure AD B2C Tenant":
<p style="text-align:center;"><img class="alignnone wp-image-240" src="/images/devisland/article6/assets/adb2c4.png?w=300" alt="" width="471" height="146" /></p>
Then provide your organization name, initial domain name and country. Click "Create" button:
<p style="text-align:center;"><img class="alignnone wp-image-242" src="/images/devisland/article6/assets/adb2c5.png?w=191" alt="" width="245" height="386" /></p>
Once AD is created you can manage it:

<img class=" wp-image-244 aligncenter" src="/images/devisland/article6/assets/adb2c6.png?w=300" alt="" width="455" height="126" />

&nbsp;
<h3><strong>Connect Azure Active Directory B2C with Azure subscription</strong></h3>
You can notice that alert about missing subscription is displayed:
<p style="text-align:center;"><img class="alignnone wp-image-247" src="/images/devisland/article6/assets/adb2c7.png?w=300" alt="" width="493" height="207" /></p>
Click on the alert to proceed. There is clear information displayed how to link AD B2C with Azure subscription:
<p style="text-align:center;"><img class="alignnone wp-image-250" src="/images/devisland/article6/assets/adb2c8.png?w=300" alt="" width="507" height="282" /></p>
Click "Switch Directories" and select directory with active Azure subscription. Once you switch, find Azure B2C under Marketplace:
<p style="text-align:center;"><img class="alignnone wp-image-236" src="/images/devisland/article6/assets/adb2c3.png?w=300" alt="" width="501" height="122" /></p>
Now select "Link an existing Azure AD B2C Tenant to my Azure subscription":
<p style="text-align:center;"><img class="alignnone wp-image-254" src="/images/devisland/article6/assets/adb2c9.png?w=300" alt="" width="510" height="63" /></p>
Choose Tenant, Subscription and optionally create new resource group (you can use existing):
<p style="text-align:center;"><img class="alignnone wp-image-256" src="/images/devisland/article6/assets/adb2c10.png?w=204" alt="" width="244" height="359" /></p>
<p style="text-align:center;"><img class="alignnone size-medium wp-image-258" src="/images/devisland/article6/assets/adb2c11.png?w=300" alt="" width="300" height="103" /></p>
Once you click "Go to resource" and then click on the square with the name of your Tenant:
<p style="text-align:center;"><img class="alignnone wp-image-260" src="/images/devisland/article6/assets/adb2c12.png?w=300" alt="" width="343" height="319" /></p>
Azure AD B2C is ready:
<p style="text-align:center;"><img class="alignnone wp-image-262" src="/images/devisland/article6/assets/adb2c13.png?w=300" alt="" width="486" height="270" /></p>

<h3></h3>
<h3><strong>Configure Sign up policy</strong></h3>
Once Azure AD B2C is ready to use it is time to configure polices. Policies fully describe consumer identity experiences such as sign-up, sign-in, or profile editing.

Example:

sign-up policy allows you to control behaviors by configuring the following settings:
<ul>
 	<li>Account types (social accounts such as Facebook or local accounts such as email addresses) that consumers can use to sign up for the application</li>
 	<li>Attributes (for example, first name, postal code, and shoe size) to be collected from the consumer during sign-up</li>
</ul>
You can read more under <a href="https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-reference-policies" target="_blank" rel="noopener">this</a> link.

<strong>Sign-up policy</strong>

Add sign-up policy. Click "Sign-up policy":
<p style="text-align:center;"><img class="alignnone wp-image-268" src="/images/devisland/article6/assets/adb2c14.png?w=283" alt="" width="205" height="217" /></p>
Click "Add" button. Blade with configuration should be displayed:
<p style="text-align:center;"><img class="alignnone wp-image-271" src="/images/devisland/article6/assets/adb2c15.png?w=145" alt="" width="189" height="391" /></p>

<ul>
 	<li>Name - field where you can type the name of the policy</li>
 	<li>Identity providers - here you can select whether you want to register user with e-mail and password or with for instance with Facebook</li>
 	<li>Sign-up attributes - you can specify which attributes should be collected during registration, like City or Surname:</li>
</ul>
<p style="text-align:center;"><img class="alignnone wp-image-274" src="/images/devisland/article6/assets/adb2c16.png?w=300" alt="" width="475" height="253" /></p>

<ul>
 	<li>Application claims - claims returned in token after successful authentication (the same like above)</li>
 	<li>Multifactor authenticaton - you can enable multi-factor authentication</li>
 	<li>Page UI customization - you can provide your custom UI for registration with Azure AD B2C</li>
</ul>
I will select collect "Display Name" during registration and from claims:

<img class=" wp-image-279 aligncenter" src="/images/devisland/article6/assets/adb2c17.png?w=300" alt="" width="395" height="158" />

and standard registration with email and password as "Identity Provider":
<p style="text-align:center;"><img class="alignnone wp-image-280" src="/images/devisland/article6/assets/adb2c18.png?w=300" alt="" width="373" height="128" /></p>
Once you click "Create" button policy should be displayed:
<p style="text-align:center;"><img class="alignnone wp-image-282" src="/images/devisland/article6/assets/adb2c19.png?w=257" alt="" width="316" height="369" /></p>

<h3><strong>Register application</strong></h3>
<p style="text-align:center;">Once policies are defined it is time to register Web API applciation. Click "Application" section and then "Add":
<img class="alignnone wp-image-285" src="/images/devisland/article6/assets/adb2c20.png?w=300" alt="" width="454" height="280" /></p>
To register Web Api application few details have to be provided:
<ul>
 	<li>Name - name of the application</li>
 	<li>Web App/Web API - turn on to indicate that you want to register Web API application</li>
 	<li>Allow implicit flow - this option should be enabled if your app needs to use OpenID Connect sign in</li>
 	<li>Reply URL - URL to which app should redirect after successful authentication. For Web API it can be just localhost as we do not have any UI</li>
 	<li>App ID URI - optional attribute to specify unique Uri to identify your application. Add "api" at the end of the address</li>
 	<li>Native client - should be disabled because we will not use mobile or desktop application</li>
</ul>
<p style="text-align:center;"><img class="alignnone wp-image-289" src="/images/devisland/article6/assets/adb2c21.png?w=268" alt="" width="360" height="402" /></p>
Once you click "Create" button application should be displayed on the list:
<p style="text-align:center;"><img class="alignnone wp-image-292" src="/images/devisland/article6/assets/adb2c22.png?w=300" alt="" width="390" height="375" /></p>
&nbsp;

Application is registered. Now its time to setup ASP .NET Core Web Api application.

&nbsp;
<h3><strong>Integrate ASP .NET Core Web API with Azure AD B2C</strong></h3>
In Visual Studio select Web Application template and then choose API:
<p style="text-align:center;"><img class="alignnone wp-image-297" src="/images/devisland/article6/assets/adb2c23.png?w=300" alt="" width="423" height="261" /></p>
<p style="text-align:center;"><img class="alignnone wp-image-299" src="/images/devisland/article6/assets/adb2c24.png?w=300" alt="" width="426" height="278" /></p>
Add "Microsoft.AspNetCore.Authentication.JwtBearer" NuGet package:

<img class=" wp-image-301 aligncenter" src="/images/devisland/article6/assets/adb2c25.png?w=300" alt="" width="432" height="141" />

Then its time to add Azure B2C settings in "appsettings.json" file:

[code language="csharp"]

"AzureAdB2C": {
"Tenant": "devisland.onmicrosoft.com",
"ClientId": "93f4e299-xxxxx-4feb-8b0c-xxxxxxxxxxx",
"Policy": "B2C_1_SignUpPolicy"
}

[/code]

Copy "Tenant", "ClientId "and "Policy" from Azure portal.

Now its time to integrate Azure AD B2C authentication in Startup.cs class.

"ConfigureServies" method should look like below:

[code language="csharp"]
 public void ConfigureServices(IServiceCollection services)
        {
            services.AddAuthentication(options =>
            {
                options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
            })
                 .AddJwtBearer(jwtOptions =>
                 {
                     jwtOptions.Authority = $"https://login.microsoftonline.com/tfp/{Configuration["AzureAdB2C:Tenant"]}/{Configuration["AzureAdB2C:Policy"]}/v2.0/";
                     jwtOptions.Audience = Configuration["AzureAdB2C:ClientId"];
                     jwtOptions.Events = new JwtBearerEvents
                     {
                         OnAuthenticationFailed = AuthenticationFailed
                     };
                 });

            services.AddMvc();
        }
[/code]

"Configure" method should look like below. "app.UseAuthentication" should be added:

[code language="csharp"]
 public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            app.UseAuthentication();

            app.UseMvc();
        }
[/code]

Last change - add "Authentication" attribute in "ValuesController" generated as default controller to prevent unauthorized access:

[code language="csharp"]
    [Authorize]
    [Route("api/[controller]")]
    public class ValuesController : Controller
[/code]

Rebuild project to check if everything was configured as expected.

&nbsp;
<h3>Test entire solution</h3>
Now its time to test Azure AD B2C authentication with ASP .NET Core Web API.

In Azure portal open "B2C_1_SignUpPolicy" policy and click "Run now" button:

<img class=" wp-image-309 aligncenter" src="/images/devisland/article6/assets/adb2c26.png?w=300" alt="" width="379" height="308" />

Registration page should be displayed. Please note that there is additional field called "Display Name" we configured in the policy:
<p style="text-align:center;"><img class="alignnone wp-image-311" src="/images/devisland/article6/assets/adb2c27.png?w=300" alt="" width="366" height="349" /></p>
Fill all required data. After successful authentication token should be returned in URL:
<p style="text-align:center;"><img class="alignnone wp-image-313" src="/images/devisland/article6/assets/adb2c28.png?w=300" alt="" width="586" height="23" /></p>
Copy the token. I use <a href="https://www.getpostman.com/" target="_blank" rel="noopener">Postman</a> to test API requests.

If I invoke "api/values" endpoint without token API will return 401 unauthorized http status:

<img class=" wp-image-316 aligncenter" src="/images/devisland/article6/assets/adb2c29.png?w=300" alt="" width="485" height="183" />

After adding token in header I am able to get values from API:
<p style="text-align:center;"><img class="alignnone wp-image-320" src="/images/devisland/article6/assets/adb2c30.png?w=300" alt="" width="490" height="198" /></p>

<h3></h3>
<h3><strong>Wrapping up</strong></h3>
In this article I presented how to configure Azure Active Directory B2C and integrate authentication in ASP .NET Core Web API project. I encourage you to test different policies setup and to integrate your Azure AD B2C with identity providers like Facebook or Google. Source code of the DevIslandWebApi project is available on my Github <a href="https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/AzureB2CWebApi" target="_blank" rel="noopener">here.</a>