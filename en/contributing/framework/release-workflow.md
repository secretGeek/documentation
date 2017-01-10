# Release Workflow

## Development

Once pull-requests have been merged into `develop` a new release is automatically generated and published to our [MyGet feed](pre-release-builds.md). There is nothing you need to do as the package identifier and library version is [automatically incremented](semantic-versioning.md) based upon our [GitVersion configuration](https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml).

![commits to develop are automatically pushed to MyGet](/images/contributing/commits-to-develop-are-automatically-pushed-to-myget.png)

## Production

