
## Prepare your release

`@TODO`

Depending of the git workflow used for the repository, you might just use the state of master for 
your new release or create a `release-x.x` branch.

Make sure you bump the version in the `VERSION` file and update the changelog.

Prometheus projects follow the [Semantic Versioning](http://semver.org/) as our versioning strategy.
So please use the `vx.x.x` pattern for stable release or `vx.x.x-rc.x` for pre-release.

## Draft the new release

There is two way to draft your new release:

* Tag manually your version

You can use the commandline to push the tag representing your release.

```bash
$ git tag -a vx.x.x
$ git push --tags
```
Signed tag with a GPG key is appreciated, so if you have added a GPG key to your Github account using the following [procedure](https://help.github.com/articles/generating-a-gpg-key/), you can add the `-S` flag to the `git tag` command to sign it with your key.

At this moment, the release process through CircleCI will be triggered for this tag.
You must create a Github Release using the UI for this tag otherwise CircleCI will not be able to upload tarballs for this tag.
Go to the releases page of the project, click on the `Draft a new release` button and select the tag you just pushed.
Usually we entitle this release like this `vx.x.x / Y-m-d`.
You can also put the changelog for this version in the description.

* Use the Github UI

You can also create the tag + Github release through the Github UI by going to the releases page and then click on the `Draft a new release` button and enter your tag version.
Usually we entitle this release like this `vx.x.x / Y-m-d`.
You can also put the changelog for this version in the description.


Now all you can do is to wait for tarballs to be uploaded to the Github release and Docker images to be pushed to the Docker Hub and Quay.io.
