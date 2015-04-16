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

So far there are a few different build workflows surfacing, involving similar functions:-

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
    
Number 1 seems like something GitTools.exe should just do based on a boolean command line param or boolean setting in a config file.

For the rest of the above, it seems likely that a typical user may want many, different build configurations for the same repo - i.e CI, Release, Publish etc (possibly using chaining in team city for example) - where each build configuration could have different combinations of the above functions.

This basically means that GitTools.exe needs to do one or both of these:

1. Needs to expose individual commands for each of the above, so that users can call it multiple times to perform just the function necessary during their build. 
2. Needs to expose a single command that takes an argument pointing to a config file - where there would be a different config per build configuration. Based on this config for example, it would decide which steps above (if any) it would execute.

For example 'Release.config'
'''
UseSemVer: true
CreateReleaseNotes: { File: ReleaseNotes.md, IssueTracker: Jira }
CreateRelease: { DescriptionFile: ReleaseNotes.md, Platform: GitHub, Artifacts: Installer.msi, NuGetPackage.nupkg, Status: Pre-Release }

'''
Then 'Publish.config'
'''
UpdateRelease: { Platform: GitHub, Tag: "%some build variable here%", Status: Published }

'''

Running GitTools.exe -Release would run against the first config file, and create release notes, and create a release in one go.
Running GitTools.exe -Publish would run against the second config file, and simply update the existing release with the given tag.

Propose using yaml for the config file format.

Propose that in the config file, can use Tokens for arguments passed in via GitTools.exe.

For example this config:-

'''
UpdateRelease: { Platform: GitHub, Tag: "%tag%", Status: Published }

'''

GitTools.exe would expect to be called with a -tag argument specifying the tagname for the release to be updated.

  
# Comments

Unclear which (if any) of the following tools belong in the command line -

* GitLink
* APIComparer
* APIApprover

Different Plugins (Issue trackers etc) have different arguments. 
We need some mechanism to surface their particular command line arguments / options via GitTools.exe.
This would require basically an extensible command line architecture - could be a little tricky. 
For example, the BitBucket releases plugin may require additional / different command line arguments than say, the Github releases plugin.
This wouldn't be too tricky to resolve, should just be routing / delegating the command arguments to the appropriate plugin for validation of the arguments.



