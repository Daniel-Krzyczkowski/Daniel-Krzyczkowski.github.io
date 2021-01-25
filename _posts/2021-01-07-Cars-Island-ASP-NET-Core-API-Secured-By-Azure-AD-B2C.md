---
title: "Cars Island ASP .NET Core API secured by Azure AD B2C - part 2"
excerpt: "This article presents how to secure ASP .NET Core API with Azure AD B2C using Microsoft Identity Web library."
header:
  image: /images/devisland/article51/assets/CarsIslandWebApiAdB2C1.PNG
---

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C1.PNG?raw=true" alt="Cars Island ASP .NET Core API secured by Azure AD B2C - part 2"/>
</p>


# Introduction

In [my previous article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the second article from the series where I would like to discuss how Cars Island ASP .NET Core Web API is secured by Azure AD B2C and how [Microsoft Identity Web library](https://www.nuget.org/packages/Microsoft.Identity.Web) is used there.

Cars Island Web API provides an Open API definition with a description of all endpoints. Endpoint responsible for handling car reservation requests requires an access token. It means that this endpoint requires the user's authorization first. This is the topic I am going to discuss in this article.

*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C2.PNG?raw=true" alt="Image not found"/>
</p>


# Azure AD B2C instance and application registration

First of all, it is worth reminding the solution architecture. As you can see below, the Azure AD B2C service is used. As I mentioned in the previous article, Azure Active Directory B2C is an identity service in the Azure cloud that enables user authentication and management. It is used in the Cars Island to authenticate users. Web portal and Web API applications are secured by Azure AD B2C. To make a new reservation for a specific car, the user has to login first. If you want to learn more about Azure AD B2C, I recommend checking the official documentation. In the Cars Island solution, I used custom policies and custom branding to adjust the look & feel of login, registration, password reset, and user profile pages.

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C3.png?raw=true" alt="Image not found"/>
</p>


## Web API application registration in the Azure AD B2C

To secure ASP .NET Core Web API application we have to register a new application in the Azure AD B2C directory first.

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C4.PNG?raw=true" alt="Image not found"/>
</p>

Once the application is created there is *Application (client) ID* value generated. It is used to uniquely identify the application in the Azure AD B2C tenant.

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C5.PNG?raw=true" alt="Image not found"/>
</p>

Then we have to expose an API to make it possible to request access token with specific scope. This is why under the *Expose an API* section we have to add new scope:


<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C6.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C7.PNG?raw=true" alt="Image not found"/>
</p>

That's it. Now we have an application registered in the Azure AD B2C and we exposed the scope to make it possible to request an access token for this specific Web API (Cars Island API).


# Microsoft Identity Web library integration

The next step is to configure authentication and authorization in the ASP .NET Core Web API application. First of all, it is worth mentioning that with new library called [Microsoft Identity Web](https://www.nuget.org/packages/Microsoft.Identity.Web) it much easier to do it. This library provides classes and methods to simplify the process of securing the application with Azure Active Directory and Azure Active Directory B2C.

To secure ASP .NET Core Web API with Azure AD B2C, add Microsoft Identity Web NuGet package. Then, in the appsettings.json file include below section:


```json
  "AzureAdB2C": {
    "Instance": "",
    "ClientId": "",
    "Domain": "",
    "SignUpSignInPolicyId": "B2C_1A_SignUpOrSignin"
  },
```

* Instance - URL of your Azure AD B2C instance, in my case this is *https://carsisland.b2clogin.com*
* ClientId - ID of the application we registered before in the Azure portal
* Domain - name of your Azure AD B2C tenant, in my case this is *carsisland.onmicrosoft.com*
* SignUpSignInPolicyId - the name of the Azure AD B2C custom policy responsible for users authentication and registration

Then with one line you can integrate authentication in the source code using the Microsoft Identity Web library extension method:

```csharp
        public static IServiceCollection AddAuthenticationWithAuthorizationSupport(this IServiceCollection services, IConfiguration config)
        {
            services.AddMicrosoftIdentityWebApiAuthentication(config, "AzureAdB2C");

            return services;
        }
```

You can see the above extension method in my GitHub repository where the project is located.

Then in the *[Startup.cs](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Startup.cs)* file we can call this method:

```csharp
 public void ConfigureServices(IServiceCollection services)
        {
            services.AddAppConfiguration(Configuration)
                    .AddAuthenticationWithAuthorizationSupport(Configuration)
            ...
        }
```

Then in the *Configure* method we have to add these two lines:

```csharp
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            ...

            app.UseAuthentication();
            app.UseAuthorization();

           ...
        }
```

In this project I decided to require authenticated users for all available endpoints. To do it in the *[Startup.cs](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Startup.cs)* file, in the *ConfigureServices* method I added below filter:

```csharp
  services.AddControllers(configure =>
      {
           ...
           var policy = new AuthorizationPolicyBuilder()
                                .RequireAuthenticatedUser()
                                .Build();
          configure.Filters.Add(new AuthorizeFilter(policy));
      });
```


With above filter applied, all enpoints requires authenticated used. To disable this rule for specific endpoints, *AllowAnonymous* attribute can be applied like I did it in the *[CarController](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Controllers/CarController.cs)* for instance:


```csharp
    [AllowAnonymous]
    [ApiController]
    [Route("api/[controller]")]
    public class CarController : ControllerBase
    ...
```

# Authorization setup

As I mentioned before, with Microsoft Identity Web library it is much easier to configure not only authentication but authorization too. In the Cars Island API, there is an endpoint responsible for handling requests related to car reservations. In the *[CarReservationController](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Controllers/CarReservationController.cs)* class there is attribute: *[Authorize(Policy = "AccessAsUser")]*. This attribute underneath verifies if the access token sent in the authorization header to the API contains a specific claim - in this case *access_as_user* claim we have described in the Azure portal above in the article.

```csharp
    [Authorize(Policy = "AccessAsUser")]
    [ApiController]
    [Route("api/[controller]")]
    public class CarReservationController : ControllerBase
    ...
```

Now let's discuss how this attribute is defined in the source code. In the API project there is a folder called *[AuthorizationPolicies](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/tree/master/src/web-api/CarsIsland.API/AuthorizationPolicies)*. This folder contains two classes:

* ScopesRequirement - this class implements *IAuthorizationRequirement* and has only one property: ScopeName. This property keeps the name of the claim that should be present in the access token sent to the API
* ScopesHandler - this class derives from *AuthorizationHandler* class and implements *HandleRequirementAsync* method. Note that this class is generic and in this specific scenario we use *ScopesRequirement* as a requirement

Below there is the source code of *ScopesHandler* class. Please note that specific requirement is succeeded only when *access_as_user* scope is presented in the access token sent to the API:

```csharp
    public class ScopesHandler : AuthorizationHandler<ScopesRequirement>
    {
        protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
                                                        ScopesRequirement requirement)
        {
            if (!context.User.Claims.Any(x => x.Type == ClaimConstants.Scope)
               && !context.User.Claims.Any(y => y.Type == ClaimConstants.Scp))
            {
                return Task.CompletedTask;
            }

            Claim scopeClaim = context?.User?.FindFirst(ClaimConstants.Scp);

            if (scopeClaim == null)
                scopeClaim = context?.User?.FindFirst(ClaimConstants.Scope);

            if (scopeClaim != null && scopeClaim.Value.Equals(requirement.ScopeName, StringComparison.InvariantCultureIgnoreCase))
            {
                // Success only when there is a specific claim presented in the access token:
                context.Succeed(requirement);
            }

            return Task.CompletedTask;
        }
    }
```

The last step is to specify which claim exactly we expect to be included in the access token. In the *[Startup.cs](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Startup.cs)* file there is below method used:

```csharp
            services.AddAuthorization(options =>
            {
                options.AddPolicy("AccessAsUser",
                        policy => policy.Requirements.Add(new ScopesRequirement("access_as_user")));
            });
```

As you can see we require *access_as_user* to be present as a claim in the access token sent to the API. If there is no *access_as_user* scope, the request will fail and *Unauthorized* response will be returned.

Below I present content of decoded JWT access token returned from the Azure AD B2C service:


<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C8.PNG?raw=true" alt="Image not found"/>
</p>

As you can see there is *access_as_user* presented.


# Authorization section in the Swagger UI

To make it easier to develop and verify if authorization works, I decided to add Authorization component to the Swagger UI:


<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C9.PNG?raw=true" alt="Image not found"/>
</p>

To enable the above functionality, in the *[SwaggerCollectionExtensions](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Core/DependencyInjection/SwaggerCollectionExtensions.cs)* class I wrote the below method where I included handling JWT bearer token:

```csharp
    public static class SwaggerCollectionExtensions
    {
        public static IServiceCollection AddSwagger(this IServiceCollection services)
        {
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "Cars Island API", Version = "v1" });

                var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
                var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
                c.IncludeXmlComments(xmlPath);

                c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    Name = "Authorization",
                    Type = SecuritySchemeType.ApiKey,
                    Scheme = "Bearer",
                    BearerFormat = "JWT",
                    In = ParameterLocation.Header,
                    Description = "JWT Authorization header using the Bearer scheme"
                });
                c.AddSecurityRequirement(new OpenApiSecurityRequirement
                {
                    {
                          new OpenApiSecurityScheme
                            {
                                Reference = new OpenApiReference
                                {
                                    Type = ReferenceType.SecurityScheme,
                                    Id = "Bearer"
                                }
                            },
                            Array.Empty<string>()
                    }
                });
            });

            return services;
        }
    }
```

Once Swagger UI is displayed, JWT token can be added.


# Summary

In this article, I described how to secure ASP .NET Core Web API with Azure AD B2C identity service using Microsoft Identity Web library. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details.

If you want to learn more about Azure AD B2C service specifically, please check below resources:

<p align="center">
<img src="/images/devisland/article51/assets/CarsIslandWebApiAdB2C10.png?raw=true" alt="Image not found"/>
</p>


