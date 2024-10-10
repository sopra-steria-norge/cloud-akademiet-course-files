# Add authentication to your API (using Microsoft Authentication Library in Swagger) - HOW TO

## Starting branch
[ef-core](https://github.com/sopra-steria-norge/WhoOwesWhat-Net48/tree/ef-core)


What we are going to do in this part of the course:
- Add middleware in Program.cs (using Swashbuckle and config to set up Swagger)
- Add configuration & variables for identity in appsettings.json
- Decoration on controllers using the `[Authorize]` filter
- Application registration in Azure (we have already created this --> [Application registration](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationMenuBlade/~/Overview/appId/a6e91cd7-badc-4ce3-8311-bf5d0ed39f3c/defaultBlade/Branding))

## TEST: Before we start doing changes, make sure that your WhoOwesWhat solution is built successfully and that the API returns 200 (OK)
- Build solution (F6) and launch app (F5)
- Try the endpoint `POST User/CreateUser` with some mock data of your choosing, should get a 200 (OK) response (since we're currently using an unprotected API)
- Make sure the `personGuid` is unique in the CreateUser request

![200 response](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/auth-images/200_ok_response_CreateUser.png)

## Add the neccessary NuGet packages to your project
- Add Nuget packages with `CLI`, `Package Manager` or right-click on solution and select `Manage NuGet Packages for Solution...`
- Required packages: `Swashbuckle.AspNetCore` (probably in your project already, install required only if not already there) & `Microsoft.Identity.Web`

![Swashbuckle.AspNetCore](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/auth-images/install_nuget_Swashbuckle.AspNetCore.png)

![Microsoft.Identity.Web](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/auth-images/install_nuget_Microsoft.Identity.Web.png)

## Use the built-in `Task list` in Visual Studio to locate the TODO's in your solution
![Task list](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/task-list.png)

## TODO: 1. Add section AzureAd to `appsettings.json`
- Add section AzureAd to `appsettings.json` and populate based on the identity we've already created --> [Application registration](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationMenuBlade/~/Overview/appId/a6e91cd7-badc-4ce3-8311-bf5d0ed39f3c/defaultBlade/Branding)

Enter the relevant application registration in the Azure Portal (Sopra Steria tenant). And copy the relevant attributes.
![App reg. with highligthed attributes](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/auth-images/app_reg_whooweswhat.png)

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
"AzureAd":{
 "Instance": "https://login.microsoftonline.com/",
 "ClientId": "", // Use the Application ID URI
 "Audience": "", // Same as ClientId in this case
 "TenantId": ""
}
```
</details>

## TODO: 2. Add authentication middleware and enable authentication for your application
<details>
  <summary> Code snippet (spoiler!) </summary>

In Program.cs add the following lines of code.
```csharp
builder.Services.AddMicrosoftIdentityWebApiAuthentication(builder.Configuration);

app.UseAuthentication();
```
</details>

 ## TODO: 3. Add decorators to controllers
<details>
  <summary> Code snippet (spoiler!) </summary>

In FriendrequestController.cs and UserController.cs add authorization filters on the top of each controller to protect all endpoints.
```csharp
[Authorize]
```
</details>

## TEST: Test the application as you did in the first step in this workshop
- Launch your app try the endpoint `POST User/CreateUser` in Swagger again (should now get a 401 (Unauthorized) response, i.e. the API is now protected by authentication.

 ## TODO: 4. Configure Swagger UI to use interactive authentication
 1. Get the relevant environment variables from AzureAd section in appsettings.json
 2. Use builder to register Service, and make the app use SwaggerUI with OAuth. In addition set the relevant options for our authentication flow
 3. Configure the HTTP request pipeline and update with relevant options

<details>
  <summary> Code snippet to get AzureAd configuration (spoiler!) </summary>

```csharp
var azureAdSection = builder.Configuration.GetSection("AzureAd");
var tenantId = azureAdSection.GetValue<string>("TenantId");
var instance = azureAdSection.GetValue<string>("Instance");
var clientId = azureAdSection.GetValue<string>("ClientId");
```
</details>

<details>
  <summary> Code snippet to add Swagger service with correct options (spoiler!) </summary>

```csharp
builder.Services.AddSwaggerGen(options =>
{
    // Add this to avoid errors in the Swagger UI
    // https://github.com/swagger-api/swagger-ui/issues/7911
    options.CustomSchemaIds(s => s.FullName?.Replace("+", "."));

    // Enabled OAuth security in Swagger
    var msalBaseUrl = $"{instance}/{tenantId}/oauth2/v2.0";
    options.AddSecurityRequirement(new OpenApiSecurityRequirement() 
    {
        {
            new OpenApiSecurityScheme {
                Reference = new OpenApiReference {
                    Type = ReferenceType.SecurityScheme,
                    Id = "oauth2"
                },
                Scheme = "oauth2",
                Name = "oauth2",
                In = ParameterLocation.Header
            },
            new List <string> ()
        }
    });
    options.AddSecurityDefinition("oauth2", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.OAuth2,
        Flows = new OpenApiOAuthFlows
        {
            Implicit = new OpenApiOAuthFlow()
            {
                AuthorizationUrl = new Uri($"{msalBaseUrl}/authorize"),
                TokenUrl = new Uri($"{msalBaseUrl}/token"),
                Scopes = new Dictionary<string, string>
	        {
	            { $"{clientId}/user_impersonation", "Access application on user behalf" }
	        }
            }
        }
    });
});
```
</details>

<details>
  <summary> Code snippet to configure Swagger UI with relevant options (spoiler!) </summary>
	
```csharp
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
	options.OAuthAppName("WhoOwesWhat");
	options.OAuthClientId(clientId);
	options.OAuthUseBasicAuthenticationWithAccessCodeGrant();
    });
}
```
</details>

## TEST: Authenticate via the padlock button in Swagger (top right)
- Launch application and authenticate using the padlock button - should now be able to log in
- Try the endpoint `POST User/CreateUser` again - should now work and give a 200 (OK) response, since you're able to get a bearer token and authenticate.
- Inspect the bearer token aquired from the application registration using the website [jwt.io](https://jwt.io/)

## Sources used in the course / best practice with regards to the topic Web API Authentication using MSAL in .NET

- MS authentication:
https://github.com/AzureAD/microsoft-identity-web/wiki/web-apis
 
- Use MS authentication with Swagger UI generated by Swashbuckle
https://www.josephguadagno.net/2022/06/03/enabling-user-authentication-in-swagger-using-microsoft-identity
