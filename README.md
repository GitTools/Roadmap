# Roadmap
First a bit of history. Particular (Simon Cropp and Andreas Öhlund) myself (Jake Ginnivan) were wanting to automate semantic version number calculation. The plan was we would build two tools GitHubFlowVersion (Jake) and GitFlowVersion (Particular). After both of these tools were working we brought them together, taking things that worked and removing things which didn't. GitVersion was the result. v3 is now configuration driven and seems to solve most of the needs of the community so it is time to start working on the next problem. Release notes and other release related things.

The glimpse guys came up with semanticreleasenotes.org a while back, and it seems that is a good release note format to standardise on. Around this space there are a bunch of open source tools (see 
[See list in the wiki](https://github.com/GitTools/Roadmap/wiki/Tools-for-Consolidation)), this repo is trying to come up with a plan as a group on where we should take all these tools and try and combine the effort to create something which works for all of us.

The basic idea is these tools should help support this workflow:

Version -> Build -> Release (including good release notes, optionally API change list and debug symbols pointing at github).

## Workflows
This really is only the versioning/release notes part and how the release is orchestrated. The API changes are not included in here at the moment

### Jakes open source workflow/Continuous delivery mode
Here is how I imagine it working in my ideal world.

1. GitVersion constantly calculating the next version, so as soon as I tag a release the next CI build has it's version bumped
2. Milestones on github are optional, can be used to group things which *must* be fixed for the next release
3. Have a create release notes build, it is triggered manually and is a pre-requisite for the release build
4. Can trigger at any time, it will generate release notes and create a draft GitHub release with all the closed issue/pull requests (can pass labels which will be filtered)
5. The draft release can be edited manually, comments added etc
6. Run the release build, it will take the artifacts build by the CI build, run the release notes build again (to include anything closed since last run). This build will push to NuGet/Chocolatey/Whatever and Tag the release (which will cause the draft to be published. As well as anything else the release needs to do. Optionally include binaries in the GitHub release

I really want one tool for calculating version, then one for managing my releasenotes/release process?

### Octopus Deploy workflow
Another really common workflow for people which I have seen while building GitVersion is the use of Octopus deploy

1. GitVersion calculates next version in continuous deployment mode (version changes every commit)
2. Each CI build is available to Octopus deploy
3. Build is released in Octopus Deploy
   - I have not used OCtopus in quite a while, will need help with where release notes and such fit in

### Particular workflow
Particular have invested a lot of time to initially create some of these tools which the community have taken and built on

### ChocolateyGUI workflow
ChocolateyGUI has been a long time user, and contributor, to GitVersion, picking up early builds and giving invaluable feedback

The current workflow looks something like this:

1. GitFlow is used as the branching strategy
2. GitVersion is used to calculate Semantic Version on each build
3. When a merge into master branch occurs (i.e. product is ready to ship a release, otherwise why are you checking into master branch), the generated semantic version number is passed to GitHubReleaseManager, which creates a draft release on GitHub containing all the closed issues for the associated milestone.  (At this point, it is assumed that a milestone exists in GitHub which matches the generated version number, and it has been kept up to date with all issues have been worked in this release)
4. Draft release is then manually inspected, and if everything is good, the Draft is published, which tags the repo with the version number, which then triggers another build
5. This time GitHubReleaseManager exports the generated Release Notes from GitHub and bundles into Application, also it closes the associated Milestone in GitHub

Full details of this process is documented [here](http://ghrm.readme.io/v0.1.0/docs/example-workflow)


## Short term plan (implementation specific)
This is just a braindump of what could happen, i want to start a discussion to get some actionable items 

1. GitReleaseNotes should start using the SemanticReleaseNotesParser for parsing from/writing to the semantic release notes
2. GitReleaseNotes should be available as a library, the library should be rather simple with the following responsibilities:
   - It can take timestamped issues from a pluggable source
   - be able to sort/group those issues into a semantic release notes model provided by SemanticReleaseNotesParser
   - Update an existing model with the new issues (append)
   - Be able to write to a pluggable output
3. Either the different input/output sources could be included in core, or kept as separate projects? Separate initially would allow tools like GitHubReleaseManager to include some from GitReleaseNotes and also provide it's own input/output sources
4. GitHubReleaseManager and GithubReleaseCreator could probably merge?
5. GitHubReleaseManager could start using the core of GitReleaseNotes, but make many more assumptions and streamline things because it just works with GitHub
6. Same with JiraCli, it could use the core of GitReleaseNotes
7. Start creating plugins/metadata runners etc to make it easier to set all this stuff up in various build servers

## Questions
 - Is one single release tool a good long term goal? With Jira/Youtrack/Tfs/BitBucket/GitHub etc support as a source, and also have support for releasing to any of them or would a shared core and a bunch of separate tools which work with just one of those issue trackers be better?
 - Do people think trying to bring all these tools into a single org which there can be a number of core contributors helping out is a good idea?
 - How do we make the setup of all this stuff approachable?
 - Should we look at going cross platform with these tools to make it easy to use them in web/ruby/anything projects on any operating system? Would NodeJS work for this?



# Summary
As I have said, this is a super rough draft but hopefully it is a good place to start conversations and see what everyones thoughts are
