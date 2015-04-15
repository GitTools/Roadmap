# Roadmap
The GitTools org hopefully will contain a bunch of projects which all are aimed at making it easy to version and release your software.  
Because most of the tools are written in .NET we are limited to windows, it would be great to get nodejs ports and open these tools/processes up to other ecosystems and operating systems. MONO could be an option if we git libgit2sharp running on mono easily.

## Current projects
 * [Semantic Release Notes](https://github.com/Glimpse/Semantic-Release-Notes) - Structured Markdown format for release notes which can be parsed into a json object structure to be output in many formats
 * [GitVersion](https://github.com/ParticularLabs/GitVersion) - Calculates the SemVer based on your git history
 * [GitReleaseNotes](https://github.com/GitTools/GitReleaseNotes) - Simple exe which connects to GitHub, Bitbucket, Jira and creates a markdown file of closed issues grouped by releases (based on git tags/timestamp of the tag)
     * [GitReleaseNotes.Web](https://github.com/GitTools/GitReleaseNotes/tree/master/src/GitReleaseNotes.Website) - A web version of GitReleaseNotes, exposing a webpage and http endpoint to generate release notes for your application. 
 * [GitHubReleaseCreator](https://github.com/dazinator/GithubReleaseCreator) - Command line tool to create GitHub releases/upload artifacts/tag etc
 * [GitHubReleaseNotes](https://github.com/Particular/GitHubReleaseNotes) - Particular's release notes generator, is open source but has been built to match the particular release workflow. It will likely stay this way.
 * [GitHubReleaseManager](https://github.com/gep13/GitHubReleaseManager) - A fork of GitHubReleaseNotes with a lot more configuration options. It is tied to GitHub milestones and github releases. 
 * [JiraCLI](https://github.com/CatenaLogic/JiraCli) - Manages releases to Jira
 * [SemanticReleaseNotesParser](https://github.com/laedit/SemanticReleaseNotesParser) - A generic parser/writer for the semantic release notes format
 * [APIComparer](https://github.com/ParticularLabs/APIComparer) - Website/console app which generate a public API changelog for each version
 * [ApiApprover](https://github.com/JakeGinnivan/ApiApprover) - Extension to ApprovalTests to create a unit tests which will require approval to any changes to the public API, designed to catch breaking API changes asap
 * [GitLink](https://github.com/GitTools/GitLink) - Sets the symbols for a release to point at GitHub to make it easy to debug the source of any open source project

As you can see there are a heap of great projects with a lot of overlap

## Workflows
This really is only the versioning/release notes part and how the release is orchestrated. The API changes are not included in here at the moment

### Jakes open source workflow/Continuous delivery mode
Here is how I imagine it working in my ideal world.

1) GitVersion constantly calculating the next version, so as soon as I tag a release the next CI build has it's version bumped
2) Milestones on github are optional, can be used to group things which *must* be fixed for the next release
3) Have a create release notes build, it is triggered manually and is a pre-requisite for the release build
4) Can trigger at any time, it will generate release notes and create a draft GitHub release with all the closed issue/pull requests (can pass labels which will be filtered)
5) The draft release can be edited manually, comments added etc
6) Run the release build, it will take the artifacts build by the CI build, run the release notes build again (to include anything closed since last run). This build will push to NuGet/Chocolatey/Whatever and Tag the release (which will cause the draft to be published. As well as anything else the release needs to do. Optionally include binaries in the GitHub release

I really want one tool for calculating version, then one for managing my releasenotes/release process?

### Octopus Deploy workflow
Another really common workflow for people which I have seen while building GitVersion is the use of Octopus deploy

1) GitVersion calculates next version in continuous deployment mode (version changes every commit)
2) Each CI build is available to Octopus deploy
3) Build is released in Octopus Deploy
   - I have not used OCtopus in quite a while, will need help with where release notes and such fit in

### Particular workflow
Particular have invested a lot of time to initially create some of these tools which the community have taken and built on

### Chocolatey workflow
Chocolatey has been a long time user and contributor to GitVersion, picking up early builds and giving invaluable feedback


## Short term plan (implementation specific)
This is just a braindump of what could happen, i want to start a discussion to get some actionable items 

1) GitReleaseNotes should start using the SemanticReleaseNotesParser for parsing from/writing to the semantic release notes
2) GitReleaseNotes should be available as a library, the library should be rather simple with the following responsibilities:
   - It can take timestamped issues from a pluggable source
   - be able to sort/group those issues into a semantic release notes model provided by SemanticReleaseNotesParser
   - Update an existing model with the new issues (append)
   - Be able to write to a pluggable output
3) Either the different input/output sources could be included in core, or kept as separate projects? Separate initially would allow tools like GitHubReleaseManager to include some from GitReleaseNotes and also provide it's own input/output sources
4) GitHubReleaseManager and GithubReleaseCreator could probably merge?
5) GitHubReleaseManager could start using the core of GitReleaseNotes, but make many more assumptions and streamline things because it just works with GitHub
6) Same with JiraCli, it could use the core of GitReleaseNotes
7) Start creating plugins/metadata runners etc to make it easier to set all this stuff up in various build servers

## Questions
 - Is one single release tool a good long term goal? With Jira/Youtrack/Tfs/BitBucket/GitHub etc support as a source, and also have support for releasing to any of them or would a shared core and a bunch of separate tools which work with just one of those issue trackers be better?
 - Do people think trying to bring all these tools into a single org which there can be a number of core contributors helping out is a good idea?
 - How do we make the setup of all this stuff approachable?
 - Should we look at going cross platform with these tools to make it easy to use them in web/ruby/anything projects on any operating system? Would NodeJS work for this?



# Summary
As I have said, this is a super rough draft but hopefully it is a good place to start conversations and see what everyones thoughts are
