![Icon](https://raw.githubusercontent.com/devlooped/GitInfo/main/assets/images/git.png) GitInfo
============

Git Info from MSBuild, C# and VB

> A fresh and transparent approach to Git information retrieval from MSBuild and Code without 
> using any custom tasks or compiled code and tools, obscure settings, format strings, etc. 

[![Build status](https://ci.appveyor.com/api/projects/status/p9e5xdd86vnfe0q8?svg=true)](https://ci.appveyor.com/project/MobileEssentials/gitinfo) 
[![Latest version](https://img.shields.io/nuget/v/GitInfo.svg)](https://www.nuget.org/packages/GitInfo)
[![Downloads](https://img.shields.io/nuget/dt/GitInfo.svg)](https://www.nuget.org/packages/GitInfo)
[![License](https://img.shields.io/:license-MIT-blue.svg)](https://opensource.org/licenses/mit-license.php)

## Usage

After installing via [NuGet](https://www.nuget.org/packages/GitInfo):

	PM> Install-Package GitInfo

By default, if the containing project is a C#, F# or VB project, a compile-time generated 
source file will contain all the git information and can be accessed from anywhere within 
the assembly, as constants in a `ThisAssembly` (partial) class and its nested `Git` static class:

	Console.WriteLine(ThisAssembly.Git.Commit);

> NOTE: you may need to close and reopen the solution in order 
> for Visual Studio to refresh intellisense and show the 
> ThisAssembly type the first time after installing the package.

By default, GitInfo will also set `$(Version)` and `$(PackageVersion)` which the .NET 
SDK uses for deriving the AssemblyInfo, FileVersion and InformationalVersion values, 
as well as for packing. This default version is formatted from the following populated 
MSBuild properties: `$(GitSemVerMajor).$(GitSemVerMinor).$(GitSemVerPatch)$(GitSemVerDashLabel)+$(GitBranch).$(GitCommit)`. 

So, straight after install and build/pack, you will get some versioning in place :).

Alternatively, you can opt-out of this default versioning by setting `GitVersion=false` 
in your project file, if you want to just leverage the Git information and/or version 
properties/constants yourself:

```xml
<PropertyGroup>
    <GitVersion>false</GitVersion>
</PropertyGroup>
```

This allows you to use the provided constants to build any versioning attributes you want, 
with whatever information you want, without resorting to settings, format strings or anything, 
just plain code:

C#:
```
[assembly: AssemblyVersion (ThisAssembly.Git.BaseVersion.Major + "." + ThisAssembly.Git.BaseVersion.Minor + "." + ThisAssembly.Git.BaseVersion.Patch)]

[assembly: AssemblyFileVersion (ThisAssembly.Git.SemVer.Major + "." + ThisAssembly.Git.SemVer.Minor + "." + ThisAssembly.Git.SemVer.Patch)]

[assembly: AssemblyInformationalVersion (
	ThisAssembly.Git.SemVer.Major + "." + 
	ThisAssembly.Git.SemVer.Minor + "." + 
	ThisAssembly.Git.Commits + "-" + 
	ThisAssembly.Git.Branch + "+" + 
	ThisAssembly.Git.Commit)]
```

F#:
```
module AssemblyInfo

open System.Reflection

[<assembly: AssemblyVersion (ThisAssembly.Git.BaseVersion.Major + "." + ThisAssembly.Git.BaseVersion.Minor + "." + ThisAssembly.Git.BaseVersion.Patch)>]

[<assembly: AssemblyFileVersion (ThisAssembly.Git.SemVer.Major + "." + ThisAssembly.Git.SemVer.Minor + "." + ThisAssembly.Git.SemVer.Patch)>]

[<assembly: AssemblyInformationalVersion (
    ThisAssembly.Git.SemVer.Major + "." + 
    ThisAssembly.Git.SemVer.Minor + "." + 
    ThisAssembly.Git.Commits + "-" + 
    ThisAssembly.Git.Branch + "+" + 
    ThisAssembly.Git.Commit)>]

do ()
```

VB:
```
<Assembly: AssemblyVersion(ThisAssembly.Git.BaseVersion.Major + "." + ThisAssembly.Git.BaseVersion.Minor + "." + ThisAssembly.Git.BaseVersion.Patch)>
<Assembly: AssemblyFileVersion(ThisAssembly.Git.SemVer.Major + "." + ThisAssembly.Git.SemVer.Minor + "." + ThisAssembly.Git.SemVer.Patch)>
<Assembly: AssemblyInformationalVersion(
    ThisAssembly.Git.SemVer.Major + "." +
    ThisAssembly.Git.SemVer.Minor + "." +
    ThisAssembly.Git.Commits + "-" +
    ThisAssembly.Git.Branch + "+" +
    ThisAssembly.Git.Commit)>
```

> NOTE: when generating your own assembly version attributes, you will need to turn off
> the corresponding assembly version attribute generation from the .NET SDK, by setting 
> the relevant properties to false: `GenerateAssemblyVersionAttribute`, 
> `GenerateAssemblyFileVersionAttribute` and `GenerateAssemblyInformationalVersionAttribute`.


MSBuild:

```
<!-- Just edit .csproj file -->  
<PropertyGroup>
    <!-- We'll do our own versioning -->
    <GitVersion>false</GitVersion>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="GitInfo" PrivateAssets="all" />
</ItemGroup>

<Target Name="PopulateInfo" DependsOnTargets="GitVersion" BeforeTargets="GetAssemblyVersion;GenerateNuspec;GetPackageContents">
    <PropertyGroup>
      <Version>$(GitSemVerMajor).$(GitSemVerMinor).$(GitSemVerPatch)$(GitSemVerDashLabel)+$(GitBranch).$(GitCommit)</Version>
      <PackageVersion>$(Version)</PackageVersion>
      
      <RepositoryBranch>$(GitBranch)</RepositoryBranch>
      <RepositoryCommit>$(GitCommit)</RepositoryCommit>
      <SourceRevisionId>$(GitBranch) $(GitCommit)</SourceRevisionId>
    </PropertyGroup>
</Target>
```

> NOTE: because the provided properties are populated via targets that need to run
> before they are available, you cannot use the GitInfo-provided properties in a 
> PropertyGroup at the project level. You can only use them from within a target that 
> in turn depends on the relevant target from GitInfo (typically, `GitVersion` as 
> shown above, if you consume the SemVer properties).

Because this information is readily available whenever you build the project, you 
never depend on CI build scripts that generate versions for you, and you can 
always compile locally exactly the same version of an assembly that was built by 
a CI server.

You can read more about this project at the 
[GitInfo announcement blog post](http://www.cazzulino.com/git-info-from-msbuild-and-code.html).

## Details

Exposes the following information for use directly from any MSBuild 
target that depends on the GitInfo target:

```
  $(GitRepositoryUrl)
  $(GitBranch)
  $(GitCommit)
  $(GitCommitDate)
  $(GitCommits)
  $(GitTag)
  $(GitBaseTag)
  $(GitBaseVersionMajor)
  $(GitBaseVersionMinor)
  $(GitBaseVersionPatch)
  $(GitSemVerMajor)
  $(GitSemVerMinor)
  $(GitSemVerPatch)
  $(GitSemVerLabel)
  $(GitSemVerDashLabel)
  $(GitSemVerSource)
  $(GitIsDirty)
```

For C#, F# and VB, constants are generated too so that the same information can be 
accessed from code:

```
  ThisAssembly.Git.RepositoryUrl
  ThisAssembly.Git.Branch
  ThisAssembly.Git.Commit
  ThisAssembly.Git.Commits
  ThisAssembly.Git.Tag
  ThisAssembly.Git.BaseTag
  ThisAssembly.Git.BaseVersion.Major
  ThisAssembly.Git.BaseVersion.Minor
  ThisAssembly.Git.BaseVersion.Patch
  ThisAssembly.Git.SemVer.Major
  ThisAssembly.Git.SemVer.Minor
  ThisAssembly.Git.SemVer.Patch
  ThisAssembly.Git.SemVer.Label
  ThisAssembly.Git.SemVer.DashLabel
  ThisAssembly.Git.SemVer.Source
  ThisAssembly.Git.IsDirty
```

Available [MSBuild properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-properties) 
to customize the behavior:

```
  $(GitVersion): set to 'false' to prevent setting Version 
                 and PackageVersion.

  $(GitThisAssembly): set to 'false' to prevent assembly 
                      metadata and constants generation.

  $(GitThisAssemblyMetadata): set to 'false' to prevent assembly 
                              metadata generation only. Defaults 
                              to 'false'. If 'true', it will also 
                              provide assembly metadata attributes 
                              for each of the populated values.

  $(ThisAssemblyNamespace): allows overriding the namespace
                            for the ThisAssembly class.
                            Defaults to the global namespace.

  $(GitRemote): name of remote to get repository url for.
                Defaults to 'origin'.

  $(GitDefaultBranch): determines the base branch used to 
                       calculate commits on top of current branch.
                       Defaults to 'main'.

  $(GitVersionFile): determines the name of a file in the Git 
                     repository root used to provide the base 
                     version info.
                     Defaults to 'GitInfo.txt'.

  $(GitInfoReportImportance): allows rendering all the retrieved
                              git information with the specified
                              message importance ('high', 
                              'normal' or 'low').
                              Defaults to 'low'.

  $(GitIgnoreBranchVersion) and $(GitIgnoreTagVersion): determines 
                            whether the branch and tags (if any) 
                            will be used to find a base version.
                            Defaults to empty value (no ignoring).

  $(GitSkipCache): whether to cache the Git information determined
           in a previous build in a GitInfo.cache for
           performance reasons.
           Defaults to empty value (no ignoring).

  $(GitCachePath): where to cache the determined Git information
				   gives the chance to use a shared location
				   for different projects. this can improve
				   the overall build time.
				   has to end with a path seperator
				   Defaults to empty value ('$(IntermediateOutputPath)').

  $(GitNameRevOptions): Options passed to git name-rev when finding
              a branch name for the current commit (Detached head). The default is
              '--refs=refs/heads/* --no-undefined --alwas'
              meaning branch names only, falling back to commit hash.
              For legacy behavior where $(GitBranch) for detached head
              can also be a tag name, use '--refs=refs/*'.
              Refs can be included and excluded, see git name-rev docs.

  $(GitTagRegex): Regular expression used with git describe to filter the tags 
                  to consider for base version lookup.
                  Defaults to * (all)
           
  $(GitBaseVersionRegex): Regular expression used to match and validate valid base versions
                          in branch, tag or file sources. By default, matches any string that 
                          *ends* in a valid SemVer2 string.
                          Defaults to 'v?(?<MAJOR>\d+)\.(?<MINOR>\d+)\.(?<PATCH>\d+)(?:\-(?<LABEL>[\dA-Za-z\-\.]+))?$|^(?<LABEL>[\dA-Za-z\-\.]+)\-v?(?<MAJOR>\d+)\.(?<MINOR>\d+)\.(?<PATCH>\d+)$'
```

## Goals

- No compiled code or tools -> 100% transparency
- Trivially added/installed via [a NuGet package](https://www.nuget.org/packages/GitInfo)
- No format strings or settings to learn
- Simple well-structured [.targets file](https://github.com/kzu/GitInfo/blob/main/src/GitInfo/build/GitInfo.targets) 
  with plain MSBuild and no custom tasks
- [Optional embedding](https://github.com/kzu/GitInfo/blob/main/src/GitInfo/build/GitInfo.AssemblyMetadata.targets) 
  of Git info in assembly metadata
- Optional use of Git info to build arbitrary assembly/file version information, both 
  [in C#](https://github.com/kzu/GitInfoDemo/blob/main/GitInfoDemo/Properties/AssemblyInfo.cs#L10) as well 
  [as VB](https://github.com/kzu/GitInfoDemo/blob/main/GitInfoDemoVB/My%20Project/AssemblyInfo.vb#L8).
- Trivially modified/improved generated code by just adjusting a 
  [C#](https://github.com/kzu/GitInfo/blob/main/src/GitInfo/build/GitInfo.cs.pp) or 
  [F#](https://github.com/kzu/GitInfo/blob/main/src/GitInfo/build/GitInfo.fs.pp) or 
  [VB](https://github.com/kzu/GitInfo/blob/main/src/GitInfo/build/GitInfo.vb.pp) template 
  included in the [NuGet package](https://www.nuget.org/packages/GitInfo)
- 100% incremental build-friendly and high-performing (all proper Inputs/Outputs in place, smart caching of Git info, etc.)


<!-- include https://github.com/devlooped/sponsors/raw/main/footer.md -->
# Sponsors 

<!-- sponsors.md -->
[![Clarius Org](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/clarius.png "Clarius Org")](https://github.com/clarius)
[![Christian Findlay](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/MelbourneDeveloper.png "Christian Findlay")](https://github.com/MelbourneDeveloper)
[![C. Augusto Proiete](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/augustoproiete.png "C. Augusto Proiete")](https://github.com/augustoproiete)
[![Kirill Osenkov](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/KirillOsenkov.png "Kirill Osenkov")](https://github.com/KirillOsenkov)
[![MFB Technologies, Inc.](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/MFB-Technologies-Inc.png "MFB Technologies, Inc.")](https://github.com/MFB-Technologies-Inc)
[![SandRock](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/sandrock.png "SandRock")](https://github.com/sandrock)
[![Eric C](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/eeseewy.png "Eric C")](https://github.com/eeseewy)
[![Andy Gocke](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/agocke.png "Andy Gocke")](https://github.com/agocke)


<!-- sponsors.md -->

[![Sponsor this project](https://raw.githubusercontent.com/devlooped/sponsors/main/sponsor.png "Sponsor this project")](https://github.com/sponsors/devlooped)
&nbsp;

[Learn more about GitHub Sponsors](https://github.com/sponsors)

<!-- https://github.com/devlooped/sponsors/raw/main/footer.md -->
