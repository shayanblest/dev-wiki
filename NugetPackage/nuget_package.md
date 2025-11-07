# 🚀 .NET NuGet Package Guide

A complete guide for creating, packaging, and publishing a .NET library as a NuGet Package.

---

## ✅ 1. Prepare Your Project

Create a new .NET class library:

```bash
dotnet new classlib -n MyLibrary
```

Add package metadata inside your `.csproj` file:

```xml
<PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <PackageId>MyLibrary</PackageId>
    <Version>1.0.0</Version>
    <Authors>Perto</Authors>
    <Company>Perto Software</Company>
    <Description>A sample .NET library packaged for NuGet</Description>
</PropertyGroup>
```

---

## 🛠️ 2. Build the Package (Pack)

Run the pack command:

```bash
dotnet build -c Release
```

Your `.nupkg` file will be generated here:

```
bin/Release/MyLibrary.1.0.0.nupkg
```

---

## 🔑 3. Get Your NuGet API Key

Go to:

[https://www.nuget.org/account/apikeys](https://www.nuget.org/account/apikeys)

Create a new API key for publishing packages.

---

## 📤 4. Publish the Package to NuGet

```bash
dotnet nuget push bin/Release/MyLibrary.1.0.0.nupkg \
--api-key <your_api_key> \
--source https://api.nuget.org/v3/index.json
```

> It may take 5–10 minutes for the package to appear publicly.

---

## 🏬 5. Publish to a Private Feed (Optional)

Example: Azure DevOps

```bash
dotnet nuget push bin/Release/MyLibrary.1.0.0.nupkg \
--source "https://pkgs.dev.azure.com/<organization>/<project>/_packaging/<feed>/nuget/v3/index.json" \
--api-key az
```

---

## 🧰 6. Useful NuGet Commands

| Purpose          | Command                                                                                       |
| ---------------- | --------------------------------------------------------------------------------------------- |
| Build package    | `dotnet pack -c Release`                                                                      |
| Publish package  | `dotnet nuget push <path>.nupkg --api-key <key> --source https://api.nuget.org/v3/index.json` |
| Add a new source | `dotnet nuget add source <url>`                                                               |
| List sources     | `dotnet nuget list source`                                                                    |
| Remove source    | `dotnet nuget remove source <name>`                                                           |

---

## 🗂️ 7. Versioning (Semantic Versioning)

```
1.0.0 = Initial release
1.1.0 = Added new features
1.1.1 = Bug fixes
```

To update the version:

```xml
<Version>1.1.0</Version>
```

Then rebuild:

```bash
dotnet pack -c Release
```

---

## 👤 Author

\*\* Mohammad Partonia \*\*\
Last Updated: 2025-11-07

