---
title: "UWP - external file system access and environment variables"
excerpt: "In this article would like to present how to access external file system from UWP application and access environment variables."
---

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess1.png?raw=true" alt="UWP - External file system access and environment variables"/>
</p>

In the one of the recent projects I had a requirement that UWP application should access the file from the external system (C: drive) and the path to this file shold be configurable using environment variables. I saw many questions in the Internet about it so in this article I present how to configure permissions and application so it can access external file system and environment variables.


## Setup application permissions

Probably you know it but just in case - UWP applications work in "sandbox" so they also have their isolated storage to keep files. If you would like to access external directories (like C:), you have to set the right permissions in the application manifest and also in the Windows 10 system.

#### Setup permissions in the application's manifest 

Open "Package.appxmanifest" file in the text editor (you can right click on the file and select "View Code").

At the top of the file in the "Package" section add below declarations:

```csharp
xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"
IgnorableNamespaces="uap mp rescap"
```
Whole section should look like below:

```csharp
<Package
  xmlns="http://schemas.microsoft.com/appx/manifest/foundation/windows10"
  xmlns:mp="http://schemas.microsoft.com/appx/2014/phone/manifest"
  xmlns:uap="http://schemas.microsoft.com/appx/manifest/uap/windows10"
  xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"
  IgnorableNamespaces="uap mp rescap">
```

Now in the "Capabilities" section add below declarations:

```csharp
<rescap:Capability Name="broadFileSystemAccess" />
<uap:Capability Name="removableStorage"/>
```

Whole section should look like below:

```csharp
  <Capabilities>
    <Capability Name="internetClient" />
    <rescap:Capability Name="broadFileSystemAccess" />
    <uap:Capability Name="removableStorage"/>
  </Capabilities>
```

#### Permissions are configured but this is not the end.

#### Setup permissions in the Windows 10 system

Besides declaration of permissions in the app manifest we have to grant access to the file system in the Windows 10:

Open "Settings" tab and select "Privacy":

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess8.png?raw=true" alt="Image not found"/>
</p>

Then select "File system":

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess9.png?raw=true" alt="Image not found"/>
</p>

In the section "Choose which apps can access your file system" select the right UWP application:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess10.png?raw=true" alt="Image not found"/>
</p>

After all above steps application will have access to the external file system.


## Setup environment variable

I mentioned before that path to the file should be stored in the environment variable and UWP application should read it

Open Windows menu and type "environment variable", you should see below:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess3.png?raw=true" alt="Image not found"/>
</p>

Now lets add new environemnt variable for the current user. Click "New" button:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess4.PNG?raw=true" alt="Image not found"/>
</p>

Provide the name and the value for the new environment variable. In this case I provided file path and file name with extension:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess6.PNG?raw=true" alt="Image not found"/>
</p>

After clicking on "OK" button, variable should be displayed on the list:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess7.PNG?raw=true" alt="Image not found"/>
</p>


## Update source code to access environment variable and file from external system

Now we have to add the source code to read environment variable value (so the path to the file) and code to access the file content.
For this article I created demo which is available on my Github [here](https://github.com/Daniel-Krzyczkowski/UniversalWindowsPlatform/tree/master/FileAccessWithEnvironmentVariables).

In the "FileService" class there is one method - "LoadFileContentUsingPathFromEnvironemntVariable":

```csharp
    public class FileService : IFileService
    {
        public async Task<string> LoadFileContentUsingPathFromEnvironemntVariable(string environmentVariableName)
        {
            if (string.IsNullOrEmpty(environmentVariableName))
            {
                throw new ArgumentNullException("Please provide the name of environment variable");
            }

            var filePathFromEnvironmentVariable = Environment.GetEnvironmentVariable(environmentVariableName);

            if (string.IsNullOrEmpty(filePathFromEnvironmentVariable))
            {
                throw new ArgumentNullException("Please provide the value of environment variable");
            }

            var textFileFromExternalPath = await StorageFile.GetFileFromPathAsync(filePathFromEnvironmentVariable);

            var fileContent = await FileIO.ReadTextAsync(textFileFromExternalPath);

            return fileContent;
        }
    }
```

First of all we have to check if the name of the environment variable is not empty. If it is empty, exception is thrown. The same for the value. Once file path is loaded from the environment variable we want to access the file content and display it in the app. First we have to get the file and next we have to read its content. Thats it.


## Display file content in the application

Once file content is returned we can display it in the application:

<p align="center">
<img src="/images/devisland/article20/assets/UwpFileSystemAccess11.png?raw=true" alt="Image not found"/>
</p>

## Summary

In this article I presented how to set permissions for the Universal Windows Platform application so it can access external file system. I also showed how to use environment variables in the source code.