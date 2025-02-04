## Release checklist

Here's a checklist for the release process.

### Leading Up To The Release

1. Talk to team about whether there are any changes which MUST go in
   this release which may cause delay.
2. Look through outstanding issues, to identify any problems that might
   be necessary to fixup before the release. Good candidates are reports
   of the project not building on different architectures or crashes.
3. Identify a good lead for each outstanding issue, and ask them about
   a fix timeline.
4. Create a milestone for the *next* release on Github, and go though
   open issues and PRs and mark accordingly.
5. Ask (via email) the most significant contributor who has not
   already named a release to name the release (use 
   `devtools/credit --verbose v<PREVIOUS-VERSION>` to find this contributor).
   CC previous namers and team.

### Preparing for -rc1

1. Check that `CHANGELOG.md` is well formatted, ordered in areas,
   covers all signficant changes, and sub-ordered approximately by user impact
   & coolness.
2. Use `devtools/changelog.py` to collect the changelog entries from pull
   request commit messages and merge them into the manually maintained
   `CHANGELOG.md`.  This does API queries to GitHub, which are severely
   ratelimited unless you use an API token: set the `GH_TOKEN` environment
   variable to a Personal Access Token from https://github.com/settings/tokens
3. Create a new CHANGELOG.md heading to `v<VERSION>rc1`, and create a link at
   the bottom. Note that you should exactly copy the date and name format from
   a previous release, as the `build-release.sh` script relies on this.
4. Update the contrib/pyln package versions: `make update-pyln-versions NEW_VERSION=<VERSION>`
5. Create a PR with the above.

### Releasing -rc1

1. Merge the above PR.
2. Tag it `git pull && git tag -s v<VERSION>rc1`. Note that you
   should get a prompt to give this tag a 'message'. Make sure you fill this in.
3. Confirm that the tag will show up for builds with `git describe`
4. Push the tag to remote `git push --tags`.
5. Announce rc1 release on core-lightning's release-chat channel on Discord.
6. Use `devtools/credit --verbose v<PREVIOUS-VERSION>` to get commits, days 
   and contributors data for release note.
7. Prepare draft release notes including information from above step, and share 
   with the team for editing.
8. Upgrade your personal nodes to the rc1, to help testing.
9. Test `tools/build-release.sh` to build the non-reproducible images & reproducible zipfile.
10. Use the zipfile to produce a [reproducible build](REPRODUCIBLE.md).

### Releasing -rc2, etc

1. Change rc1 to rc2 in CHANGELOG.md.
2. Update the contrib/pyln package versions: `make update-pyln-versions NEW_VERSION=<VERSION>`
3. Add a PR with the rc2.
4. Tag it `git pull && git tag -s v<VERSION>rc2 && git push --tags`
5. Announce tagged rc release on core-lightning's release-chat channel on Discord.
6. Upgrade your personal nodes to the rc2.

### Tagging the Release

1. Update the CHANGELOG.md; remove -rcN in both places, update the date and add title and namer.
2. Update the contrib/pyln package versions: `make update-pyln-versions NEW_VERSION=<VERSION>`
3. Add a PR with that release.
4. Merge the PR, then:
   - `export VERSION=0.9.3`
   - `git pull`
   - `git tag -a -s v${VERSION} -m v${VERSION}`
   - `git push --tags`
5. Run `tools/build-release.sh` to build the non-reprodicible images
   and reproducible zipfile.
6. Use the zipfile to produce a [reproducible build](REPRODUCIBLE.md).
7. To create and sign checksums, start by entering the release dir: `cd release`
8. Create the checksums for signing: `sha256sum * > SHA256SUMS`
9. Create the first signature with `gpg -sb --armor SHA256SUMS`
10. The tarballs may be owned by root, so revert ownership if necessary:
    `sudo chown ${USER}:${USER} *${VERSION}*`
11. Upload the resulting files to github and save as a draft.
    (https://github.com/ElementsProject/lightning/releases/)
12. Ping the rest of the team to check the SHA256SUMS file and have them send their
    `gpg -sb --armor SHA256SUMS`.
13. Append the signatures into a file called `SHA256SUMS.asc`, verify
    with `gpg --verify SHA256SUMS.asc` and include the file in the draft
    release.
14. `make pyln-release` to upload pyln modules to pypi.org.  This requires keys
    for each of pyln-client, pyln-proto, and pyln-testing accessible to poetry.
    This can be done by configuring the python keyring library along with a
    suitable backend.  Alternatively, the key can be set as an environment
    variable and each of the pyln releases can be built and published
    independently:
    - `export POETRY_PYPI_TOKEN_PYPI=<pyln-client token>`
    - `make pyln-release-client`
    - ... repeat for each pyln package.

### Performing the Release

1. Edit the GitHub draft and include the `SHA256SUMS.asc` file.
2. Publish the release as not a draft.
3. Announce the final release on core-lightning's release-chat channel on Discord.
4. Send a mail to c-lightning and lightning-dev mailing lists, using the
   same wording as the Release Notes in github.
5. Write release blog post and announce the release on Twitter.

### Post-release

1. Look through PRs which were delayed for release and merge them.
2. Close out the Milestone for the now-shipped release.
3. Update this file with any missing or changed instructions.
