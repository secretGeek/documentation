# Release Workflow

## Development

Once pull-requests have been merged into `develop` a new release is automatically generated and published to our [MyGet feed](pre-release-builds.md). There is nothing you need to do as the package identifier and library version is [automatically incremented](semantic-versioning.md) based upon our [GitVersion configuration](https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml).

![commits to develop are automatically pushed to MyGet](/images/contributing/commits-to-develop-are-automatically-pushed-to-myget.png)

## Production

Create a pull-request from `develop` to `master`
Adjust the commit message and if it is a breaking release add the gitversion convention.
Merge the pull-request using the "squash commit" option.
Watch the build, once it completes you a draft release will appear in GitHub.
Edit the release notes.
Press publish release.
Wait for the final build to happen.
Verify packages were published to NuGet.

![](/en/images/contributing/click-edit-vnext-milestone-button.png)

![](/en/images/contributing/commits-to-master-do-not-automatically-publish-to-nuget.png)

![](/en/images/contributing/rename-vnext-milestone-to-release-version.png)

![](/en/images/contributing/ensure-all-issues-assigned-to-a-milestone-are-labeled.png)

![](/en/images/contributing/tagged-releases-automatically-publish-to-nuget.png)

![](/en/images/contributing/view-vnext-milestone.png)

![](/en/images/contributing/create-new-vnext-milestone.png)



When creating a new release by pull-requesting between `develop` and `master` always use a **merge commit pull-request**. If the release contains breaking changes then the `BREAKING` identifier can be automatically incremented by adding `+semver: breaking` on a blank line of your commit message.
