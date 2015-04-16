# Architecture Idea

For reference, the current list of tools - seeking some sort of "grand unification"? https://github.com/GitTools/Roadmap/wiki/Tools-for-Consolidation

## One (single) Command Line exe - aka GitTools.exe.

This will give end users one exe to work with when setting up their automation (Team City, Octopus etc etc) - no matter which features they need. Simpler than having to reference and configure multiple exe's.

## Pluggable / Extensible Core. 

The core would contain pure Git related functionality with no direct dependencies on any third party services (github / jira etc) - although code in the Core can "resolve" plugins at runtime in order to integrate with issue trackers / third party services etc etc.

*This may mean some sort of startup process, where available plugins are loaded and registered (in some container)*

GitVersion functionality could be contained in the Core.
GitReleaseNotes could be contained in the Core, although specific IssueTracker integration could be moved into a Plugin model (and plugins resolved at runtime)

### Additional Benefits

* Community *may* find it easier to contribute specific Plugins (with a well documented plugin model) than make changes to core code. 
* Edge cases that we cannot concieve right now may be able to be covered by custom plugin implementation in the future (extensibility is never to be laughed at)

### Concerns

* Adds complexity
* Could be overkill if no one ever develops any plugins etc.

## Plugins
We could implement the following as plugins for GitTools.exe:

* Issue trackers:
    * GitHub
    * Bitbucket
    * Jira

* Release Creation
    * GitHub
    * BitBucket
    * Jira

* Build / CI Workflow tasks
    * More on these below.

## Build / CI Workflows

So far there are a few different build workflows surfacing, all involving similar functionality:-

1. Calculate SemVer Version Numbers (must happen at start of a build)
    * Expose SemVer version Numbers to CI / Build system / MSBuild Variables etc
2. Create Release Notes (including resolving and using the relevant Issue tracker/s)
3. Creating a Release on a third party platform, including:-
    * setting the release description (release notes)
    * resolving and using the relevant third party API - GitHub, BitBucket etc
    * uploading release artifacts
    * setting the release status (draft / published etc)
4. Updating a Release on a third party platform, including:
    * modifying the release description (release notes)
    * adding / removing release artifacts
    * modifying the release status (draft / published etc)
    
Number 1 seems like something GitTools.exe should just do whever its invoked (based on a boolean command line param or boolean setting in a config file or something).

It seems likely that the user may want many, different build configurations for the same repo - i.e:

* CI
* Release
* Publish

Each build configuration would may require GitVersion to perform different combinations of the above functions.

This basically means that GitTools.exe needs to either (or both):

1. Expose individual commands ina granular way, so that users can call GitTools.exe multiple times as part of their builds, each time just performing the function necessary.
2. Expose a config mechanism. Invoke GitTools.exe passing an argument specifying a config file to use. You could then have different config files per build configuration. GitTools.exe would then interpret the config file, and decide what needs to be run.

Example 'Release.config'
```
UseSemVer: true
CreateReleaseNotes: { File: ReleaseNotes.md, IssueTracker: Jira }
CreateRelease: { DescriptionFile: ReleaseNotes.md, Platform: GitHub, Artifacts: Installer.msi, NuGetPackage.nupkg, Status: Pre-Release }
```

Example 'Publish.config'
```
UpdateRelease: { Platform: GitHub, Tag: "%some build variable here%", Status: Published }
```

Running GitTools.exe -Release would run against the first config file, which would executr GitVersion, and Release notes funcitonality - and would also Create a release on GitHub (assuming the GitHub plugin was available at runtime).
Running GitTools.exe -Publish would run against the second config file, and simply update the existing release on GitHub that has the specified tag.

Propose using yaml for the config file format.

Propose that in the config file, can use Tokens to represent arguments that are passed in via GitTools.exe. For example, with this config:

```
UpdateRelease: { Platform: GitHub, Tag: "%tag%", Status: Published }
```

You would have to have specified a -tag argument when calling GitTools.exe:

```
GitTools.exe -tag "v1.2.3"
```
# Comments

Unclear which (if any) of the following tools belong in the command line -

* GitLink
* APIComparer
* APIApprover

Different Plugins (Issue trackers etc) have different arguments. 
A mechanism would be needed to surface plugin specific command line arguments / options via GitTools.exe.
For example, if a valid MsExcelIssueTracker plugin dll was dropped in the bin directory, GitTools.exe ? should be able to list the arguments required by that plugin for example -MsExcelFile "some file.xls"
Perhaps worth looking at something like this: https://github.com/TypecastException/ConsoleApplicationBase

