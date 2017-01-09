# Semantic Versioning

Semantic Versioning is all about releases, not builds. This means that the version should only increase after only increases after you release, this directly conflicts with the concept of published CI builds. 

When you release the next version of your library/app/website/whatever you should only increment major/minor or patch then reset all lower parts to 0, for instance given 1.0.0, the next release should be either 2.0.0, 1.1.0 or  1.0.1. Bumping one of the version components by more than 1 in a single release means you will have gaps in your version number, which defeats the purpose of SemVer.

Because of this, GitVersion works out what the next SemVer of your app is on each commit. When you are ready to release you simply deploy the latest built version and tag the release it was from. This practice is called continuous delivery. GitVersion will increment the metadata for each build so you can tell builds apart. For example 1.0.0+5 followed by 1.0.0+6. It is important to note that build metadata is not part of the semantic version, it is just metadata!.

All this effectively means that GitVersion will produce the same version NuGet package each commit until you tag a release.

This causes problems for people as NuGet and other package managers do not support multiple packages with the same version with only different metadata. There are a few ways to handle this problem depending on what your requirements are:

1. GitFlow

If you are using GitFlow then builds off the develop branch will actually increment on every commit. This is known in GitVersion as continuous deployment mode. By default develop builds are tagged with the alpha pre-release tag. This is so they are sorted higher than release branches.

ReactiveUI uses [Semantic Versioning (SemVer)](http://semver.org/), adopting the use of major.minor.patch versioning, using the various parts of the version number to describe the degree and kind of change.


If the current commit is tagged, the tag is used and build metadata (Excluding commit count) is added. The other two steps will not execute
A set of strategies are evaluated to decide on the base version and some metadata about that version. These strategies include HighestReachableTag, NextVersionInConfig, MergedBranchWithVersion, VersionInBranchName etc.
The highest base version is selected, using that base version the new version is calculated.


Adding +semver: breaking or +semver: major will cause the major version to be increased,  +semver: feature or +semver: minor will bump minor and +semver: patch or +semver: fix will bump the patch.


https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml


### Versioning Form

MAJOR.MINOR.PATCH[-ALPHA-BUILDNUMBER]

### Decision Tree

MAJOR when:
  - drop support for a platform
  - adopt a newer MAJOR version of an existing dependency 
  - disable a compatibility quirk off by default

MINOR when:
  - add public API surface area 
  - add new behavior
  - adopt a newer MINOR version of an existing dependency
  - introduce a new dependency 
  
PATCH when:
  - make bug fixes
  - add support for a newer platform
  - adopt a newer PATCH version of an existing dependency
  - any other change (not otherwise captured)

When determining what to increment when there are multiple changes, choose the highest kind of change.
