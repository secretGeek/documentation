# Release Workflow

## Development

Once pull-requests have been merged into `develop` a new release is automatically generated and published to our [MyGet feed](pre-release-builds.md). There is nothing you need to do as the package identifier and library version is [automatically incremented](semantic-versioning.md) based upon our [GitVersion configuration](https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml).

![commits to develop are automatically pushed to MyGet](/images/contributing/commits-to-develop-are-automatically-pushed-to-myget.png)

## Production




When creating a new release by pull-requesting between `develop` and `master` always use a **merge commit pull-request**. If the release contains breaking changes then the `BREAKING` identifier can be automatically incremented by adding `+semver: breaking` on a blank line of your commit message.
