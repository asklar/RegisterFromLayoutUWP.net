# How to register a .net UWP app to run from layout 

## Motivation
Visual Studio is able to deploy a UWP app from layout (i.e. without installing the APPX). This has several benefits for inner-loop development: installing an APPX requires elevation to pre-install the certificate that was used to sign the APPX with, which is unnecessary and cumbersome for Debug builds while you're writing the app.

This workflow works well in VS, however the command-line experience is broken. There are docs for [Add-AppxPackage](https://docs.microsoft.com/en-us/powershell/module/appx/add-appxpackage?view=win10-ps) which tell us how to register a UWP app from layout. This seems to work for native apps but it  doesn't work for .net UWP apps: **apps will crash on launch**. The symptom is that the desktop .net framework gets loaded instead of the .net framework for UWP.

## Instructions
This project describes how to set up your project to allow deploying from layout.

1) Add the following to your `csproj`:

```xml
  <Import Project="$(OutputPath)\$(AssemblyName).Build.appxrecipe" Condition="Exists('$(OutputPath)\$(AssemblyName).Build.appxrecipe')" />
  <Target Name="Deploy">
    <Error Condition="!Exists('$(OutputPath)\$(AssemblyName).Build.appxrecipe')" Text="You must first build the project before deploying it" />
    <Copy SourceFiles="%(AppxPackagedFile.Identity)" DestinationFiles="$(OutputPath)Appx\%(AppxPackagedFile.PackagePath)" />
    <Copy SourceFiles="%(AppXManifest.Identity)" DestinationFiles="$(OutputPath)Appx\%(AppxManifest.PackagePath)" Condition="'%(AppxManifest.SubType)'!='Designer'"/>
    <Exec Command="powershell -Command Add-AppxPackage -Register $(OutputPath)Appx\AppxManifest.xml" ContinueOnError="false" />
  </Target>
```
2) Build your app in the command line as usual, e.g. `msbuild /restore /p:Configuration=Debug /p:Platform=x86`
3) Then run the `deploy` target:  `msbuild /p:Configuration=Debug /p:Platform=x86 /t:Deploy`

Done!
