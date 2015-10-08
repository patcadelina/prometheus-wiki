The following instructions describe how to cut a new release of Prometheus when the release version is the HEAD of the `master` branch. The process for cutting bugfix releases based off the latest stable release (non-HEAD) is not clearly defined yet.

1. Consider updating all or some of the vendored dependencies before the
   release. Make sure that https://github.com/prometheus/client_golang is
   vendored at a release version.
1. Create a PR that updates the `CHANGELOG.md` along with the `version/VERSION` file and get it merged.
1. Tag the new release and push the tag to GitHub:

   ```bash
   git tag $VERSION
   git push --tags
   ```

1. Head to https://github.com/prometheus/prometheus/releases and create a new release for the pushed tag.
1. Attach the relevant changelog excerpt to the release on GitHub.
1. Build `linux-amd64`, `linux-386`, and `darwin-amd64` binaries via `make tarball` and attach them to the release. Please note that this still needs some manual care, as cross-compiling is not yet supported by the Makefile.
1. Update the `stable` branch to point to the new release (to tag the latest release image on https://registry.hub.docker.com/u/prom/prometheus/):

   ```bash
   git branch -f stable <new-release-tag>
   git push origin stable
   ```

1. Head to https://hub.docker.com/r/prom/prometheus/~/settings/automated-builds/ and create a automated build associations for the new tag. Then, click "Trigger a build". Note that multiple tags on the Docker hub will need updating, depending on whether it's a patch/minor/major release. Update the `<major-version>` tag name to point to the new release with that major version. Similarly, update the `<major-version>.<minor-version>` tag. Finally, add a new tag for the full new release version.
1. Merge any release-specific documentation changes in https://github.com/prometheus/docs (changes that are only applicable to the next release should be kept in a `next-release` branch there).
1. Announce the release on the mailing list and on Twitter.