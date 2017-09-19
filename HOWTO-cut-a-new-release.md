These instructions are currently valid for the Prometheus server, i.e. the [prometheus/prometheus repository](https://github.com/prometheus/prometheus). Applicability to other Prometheus repositories depends on the current state of each repository. We aspire to use release procedures as similar as possible.

## Branch management and versioning strategy

We use [Semantic Versioning](http://semver.org/).

We maintain a separate branch for each minor release, named `release-<major>.<minor>`, e.g. `release-1.1`, `release-2.0`. The `stable` branch is supposed to point to the latest stable release.

The usual flow is to merge new features and changes into the master branch and to merge bug fixes into the latest release branch. Bug fixes are then merged into master from the latest release branch. The master branch should always contain all commits from the latest release branch.

If a bug fix got accidentally merged into master, cherry-pick commits have to be created in the latest release branch, which then have to be merged back into master. Try to avoid that situation.

Maintaining the release branches for older minor releases happens on a best effort basis. 

## Prepare your release

For a patch release, work in the branch of the minor release you want to patch.

For a new major or minor release, create the corresponding release branch based on the master branch.

Bump the version in the `VERSION` file and update `CHANGELOG.md`. It's usually a good idea to do this in a proper PR as this gives others the opportunity to chime in on the release in general and on the addition to the changelog in particular.

Note that `CHANGELOG.md` should only document changes relevant to users of Prometheus, including external API changes, performance improvements, and new features. Do not document changes of internal interfaces, code refactorings and clean-ups, changes to the build process, … Whoever is interested in those is supposed to look at the Git commit history.

## Draft the new release

Tag the new release with a tag named `v<major>.<minor>.<patch>`, e.g. `v2.1.3`. Note the `v` prefix.

You can do the tagging on the commandline:

```bash
$ git tag -a vx.y.z -m 'vx.y.z'
$ git push --tags
```
Signed tag with a GPG key is appreciated, so if you have added a GPG key to your Github account using the following [procedure](https://help.github.com/articles/generating-a-gpg-key/), you can replace the `-a` flag by `-s` flag of the `git tag` command to sign it with your key.

Once a tag is created, the release process through CircleCI will be triggered for this tag.
You must create a Github Release using the UI for this tag otherwise CircleCI will not be able to upload tarballs for this tag. __Also, you must create the Github Release using a Github user that has granted access rights to CircleCI.__ If you did not or cannot grant those rights to your personal account, you can log in as `prombot` in an anonymous browser tab. (This will, however, prevent verified releases signed with your GPG key. For verified releases, the signing identity must be the same as the one creating the release.)

Go to the releases page of the project, click on the _Draft a new release_ button and select the tag you just pushed. The title of the release is formatted `x.y.z / YYYY-MM-DD`. Add the relevant part of `CHANGELOG.md` as description. Click _Save draft_ rather than _Publish release_ at this time. (This will prevent the release being visible before it has got the binaries attached to it.)

You can also create the tag and the Github release in one go through the Github UI by going to the releases page and then click on the `Draft a new release` button and enter your tag version.

Now all you can do is to wait for tarballs to be uploaded to the Github release and Docker images to be pushed to the Docker Hub and Quay.io. Once that has happened, click _Publish release_, which will make the release publicly visible and create a GitHub notification.

## Wrapping up

If the release has happened in the latest release branch, merge the changes into master.

Point the `stable` branch to the new release.

The `prometheus/docs` repo has a branch `next-release` for documentation of new features of the Prometheus server not yet released. After you have released them, merge the branch into the master branch.

Once the binaries have been uploaded, announce the release on `prometheus-users@googlegroups.com`. Start the subject with `[ANN]`. Check out previous announcement mails for inspiration.

Finally, you have to trigger a rebuild of the docs site on [Travis-CI](https://travis-ci.org/prometheus/docs) so that the downloads page shows the newly built binaries. (Hit _Restart build_ on the details page of the current build.)

## Pre-releases

For risky or invasive changes, you might want to cut a pre-release or release candidate. The following changes to the above procedures apply:
* In line with [Semantic Versioning](http://semver.org/), append something like `-rc1` to the version (with the corresponding changes to the tag name, the release name etc.).
* Tick the _This is a pre-release_ box when drafting the release in the Github UI.
* Do not point the `stable` branch to your release tag.
* Still update `CHANGELOG.md`, but when you cut the final release later, merge all the changes from the pre-releases into the one final update.
