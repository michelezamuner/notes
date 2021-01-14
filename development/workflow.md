# Workflow

## Feature branch workflow / GitHub workflow

The core idea of the *feature branch workflow*, also known as *GitHub workflow*, is that all feature development should happen in a dedicated branch, in order to prevent development from disturbing the main codebase. This means that `master` will always contain working and tested code, which helps with continuous integration.

If development always happens on a dedicated branch, all changes can go through the pull request process, which promotes discussions around the development.

All feature branches are created off the latest `master`, and merged back to `master` as a result of the pull request.

This flow has the following disadvantages:

- if there are several releases or versions that need to be maintained, we don't have a separate branch for each release
- if the release process requires a lot of tweaking and fixes, that consumes quite some time, all the rest of the development must be stopped in the meanwhile, because we don't have a dedicated branch

## Gitflow

The *Gitflow workflow* defines a branching model based on the project release, and it's best suite for projects that have a scheduled release cycle, or that implement continuous delivery.

The Gitflow workflow also uses separate branches for every feature, but in addition to these it also uses specific branches to maintain releases.

In addition to the `master` branch, with Gitflow we also have a `develop` branch, which serves as an integration branch for features. Additionally, all commits to `master` should be tagged with a version number.

Feature branches are branched off of `develop`, and pull requests merge back to `develop`. Once `develop` has acquired enough features for a release, or the release date is approaching, a new `release` branch is forked off of `develop`: the `release` branch doesn't accept any new development, except for bugfixes or release-oriented tasks, while normal development can proceed in the `develop` branch. Once the project status in `release` is ready for release, the `release` branch is merged into `master` and tagged with a version number, and also merged back into `develop`.

Using a dedicated `release` branch allows one team to polish the project before the release, and other teams to keep working on the main `develop` branch. We can also see the various stages of development, including each release phase, clearly in the repository structure.

When a bug is discovered, or some other quick fix is necessary in a release, we create a `hotfix` branch off of `master` where we implement and test the fix, and then we merge back into `master` and `develop`. Again this allows to dedicate one team to fixing the production bug, without disturbing the others that work on the regular `develop` branch.

Using dedicated branches for the various release stages, provides these benefits:

- we can leave current development open while preparing a specific release, without worrying to ship half-baked features in it
- we can keep working on new features without worrying that they will end up in the release, if they weren't intended to
- we can quickly fix production bugs without having to finish the current development before doing it

While there are some drawbacks:

- if releases are not merged quickly enough, there might be conflicts when merging releases back into `develop`
- it's not easy to enforce the discipline required to work with the different branches the way they're intended to
- there's a lot of boilerplate work with handling all different branches
- it's easy to forget to keep all fundamental branches up to date on every change, so `master` and `develop` risk to diverge with time
- all features added to `develop` will end up in the next release, so it's not easy to work on features that should not be included in the next release

## GitLab flow

Modern development embraces Continuous Delivery, which means that code in the default branch is always deployable, and deployed. Since a set of development techniques like peer review, automated testing and static analysis can ensure that the code is always deliverable once the peer reviews are accepted, there's no need to have either dedicated release branches, nor a dedicated `develop` branch, with all their ceremonies.

However, even if the code in `master` is guaranteed to work, there might be issues of other kinds that prevent it from being actually deployed: for example the deployment needs to wait for an external approval, like with mobile applications that need to be accepted by the third party store, or we need to wait for a specific time window in order to perform the deployment. If development happens directly in `master`, these issues will halt development while waiting for the deployment.

A good solution to this problem is to still add other branches in addition to `master`, like `pre-production` and `production`, which are updated from `master` in preparation of releases, and are only used to feed deployment pipelines, meaning that now development is ever done on those branches, not even for hotfixes.

With this structure, we can configure our CI tool to automatically perform a deploy to a specific environment whenever a specific branch is updated. For example, we can link the `staging` branch to the staging environment, so that every time a push is performed to `staging`, the CI tool automatically deploys that branch to the staging server.

We can also create release branches to keep track of different versions, again following the rule that no development can be done on that branch, but all development should go through `master`, and then can be merged on the target branch.

The end result is a repository structure where there is no merge-back, but only merge-forward: this prevents developers from forgetting to merge changes back to all branches needed.

## References

- https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow
- https://guides.github.com/introduction/flow/
- https://www.freshconsulting.com/git-development-workflows-git-flow-vs-github-flow/
- https://docs.gitlab.com/ee/topics/gitlab_flow.html
