# Release Workflow

## Development

Once pull-requests have been merged into `develop` a new release is automatically generated and published to our [MyGet feed](pre-release-builds.md). There is nothing you need to do as the package identifier and library version is [automatically incremented](semantic-versioning.md) based upon our [GitVersion configuration](https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml).

![commits to develop are automatically pushed to MyGet](/images/contributing/commits-to-develop-are-automatically-pushed-to-myget.png)

## Production

By design, no single person can release a new version of ReactiveUI with-out approval from [one of the other contributors](https://github.com/orgs/reactiveui/teams/contributors). The process is kicked off by one of the contributors opening up a pull-request from `develop` to `master`

![](/en/images/contributing/create-a-pull-request-from-develop-to-master.png)

At this stage one of contributor that is proposing a new release must perform a few administrative tasks. It's the responsibility of the release approver to verify that these tasks have been performed correctly. 

![](/en/images/contributing/pull-request-review-required.png)

Edit the vNext milestone and change it to the version number of the next release. If this release contains a [breaking change](semantic-versioning.md) then increase the `BREAKING` by one and reset the `MINOR` back to zero. Otherwise just bump the `MINOR` by one. 

![](/en/images/contributing/click-edit-vnext-milestone-button.png)

If process for [merging individual pull-requests](merging-pull-requests.md) was followed perfectly then you won't need to do much here but in both parties need to verify that all issues that have been assigned to a milestone:

* Has a prefix which classifies the type of change and clearly explained what was changed because our release notes are automatically generated from this information.
* Have at _at least one or more_ label(s) assigned to the GitHub issue which classifies the scope of change. GitReleaseManager will fail to automatically generate the release notes and the draft release in GitHub if any issue has no labels.

![](/en/images/contributing/ensure-all-issues-assigned-to-a-milestone-are-labeled.png)

Once the release has been signed off, you'll need to switch your merge mode to `Create a merge commit` aka `Merge pull request` by using the little arrow on the right hand side.

![](/en/images/contributing/merge-commit-option.png)

Mash the merge pull request button and customize the merge commit message. If the release contains breaking changes then bump the `BREAKING` identifier by adding `+semver: breaking` on a blank line of your commit message.

![](/en/images/contributing/merge-commit.png)

Merging this pull-request into `master` will kick off a new build on AppVeyor but the build pipeline is configured not to publish packages to NuGet.org unless the release has been tagged.

![](/en/images/contributing/commits-to-master-do-not-automatically-publish-to-nuget.png)

Once this build completes, a new draft release with automatically generated release notes will be available for you to polish and add any flair as needed.

$INSERT IMAGE OF DRAFT RELEASE HERE.

Pressing the publish release button will stamp the repository with a git tag


Press the publish release button which will stamp the repository with the repository with the tag. 

a git-tag with the version number. 

![](/en/images/contributing/tagged-releases-automatically-publish-to-nuget.png)




![](/en/images/contributing/pull-request-into-master-then-publish-tag-to-release.png)




When creating a new release by pull-requesting between `develop` and `master` always use a **merge commit pull-request**. If the release contains breaking changes then the `BREAKING` identifier can be automatically incremented by adding `+semver: breaking` on a blank line of your commit message.


- Adjust the commit message and if it is a breaking release add the gitversion convention.
- Merge the pull-request using the "squash commit" option.
- Watch the build, once it completes you a draft release will appear in GitHub.
- Edit the release notes.
- Press publish release.
- Wait for the final build to happen.
- Verify packages were published to NuGet.

![](/en/images/contributing/click-edit-vnext-milestone-button.png)

![](/en/images/contributing/commits-to-master-do-not-automatically-publish-to-nuget.png)

![](/en/images/contributing/rename-vnext-milestone-to-release-version.png)

![](/en/images/contributing/ensure-all-issues-assigned-to-a-milestone-are-labeled.png)


![](/en/images/contributing/view-vnext-milestone.png)

![](/en/images/contributing/create-new-vnext-milestone.png)




![](/en/images/contributing/create-new-vnext-milestone.png)



