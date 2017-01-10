# Submitting A Pull Request

![](/en/images/contributing/branch-and-pull-based-workflow.png)


![Pull-request suffix](/images/contributing/semver-pull-request-into-develop.png)

Builds from pull-requests have a suffix of `pullrequest$GitHubIssueNumber` and are not automatically published to NuGet or MyGet but the packages are available for download from AppVeyor which allows the team to test the unit of change without merging into `develop`.

When merging pull-requests into the `develop` branch use a rebase and squash commit that follows our template.

