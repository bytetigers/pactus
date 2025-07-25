# Release Process

To ensure a successful release and publication of the Pactus software, it is essential to follow these key steps.
Please carefully follow the instructions provided below:

## 1. Preparing Your Environment

Before proceeding with the release process,
ensure that your `origin` remote is set to `git@github.com:pactus-project/pactus.git`
and not your local fork.

```bash
git remote -vv
```

It is recommended to re-clone the project in a location other than your current working directory.
Also, make sure that you have set up GPG for your GitHub account.

## 2. Fetch the Latest Code

Ensure that your local repository is up-to-date with the Pactus main repository:

```bash
git checkout main
git pull
```

## 3. Set Environment Variables

Create environment variables for the release version, which will be used in subsequent commands throughout this document.
Keep your terminal open for further steps.

```bash
PRV_VER="1.8.0"
CUR_VER="1.9.0"
NEXT_VER="1.10.0"
BASE_BRANCH="main"
TAG_NAME="v${CUR_VER}"
TAG_MSG="Version ${CUR_VER}"
```

## 4. Update the Version

Clear Meta and set Alias in [version.json](../version/version.json).

## 5. Update Changelog

Use [Commitizen](https://github.com/commitizen-tools/commitizen) to update the CHANGELOG. Execute the following command:

```bash
cz changelog --incremental --unreleased-version ${TAG_NAME}
perl -i -pe "s/## v${CUR_VER} /## [${CUR_VER}](https:\/\/github.com\/pactus-project\/pactus\/compare\/v${PRV_VER}...v${CUR_VER}) /g" CHANGELOG.md
perl -i -pe "s/\(#([0-9]+)\)/([#\1](https:\/\/github.com\/pactus-project\/pactus\/pull\/\1))/g" CHANGELOG.md
```

Occasionally, you may need to make manual updates to the [CHANGELOG](../CHANGELOG.md).

## 6. Create a Release PR

Generate a new PR against the base branch.
It's better to use [GitHub CLI](https://github.com/cli/cli/) to create the PR, but manual creation is also an option.

```bash
git checkout -b releasing_${CUR_VER}
git commit -a -m "chore(release): releasing version ${CUR_VER}"
git push origin HEAD
gh pr create --title "chore(release): releasing version ${CUR_VER}" --body "Releasing version ${CUR_VER}" --base ${BASE_BRANCH}
```

Wait for the PR to be approved and merged into the main branch.

## 7. Tagging the Release

Create a Git tag and sign it using your [GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification) with the following commands:

```bash
git checkout ${BASE_BRANCH}
git pull
git tag -s -a ${TAG_NAME} -m "${TAG_MSG}"
```

Inspect the tag information:

```bash
git show ${TAG_NAME}
```

## 8. Push the Tag

Now, push the tag to the repository:

```bash
git push origin ${TAG_NAME}
```

Pushing the tag will automatically create a release tag and build the binaries.

## 9. Bump the Version

Update the version inside [version.json](../version/version.json), add `beta` to the `meta` field, and:
- If this is a **major release**, update the versions inside [this document](./releasing.md#3-set-environment-variables) in step 4 and
update the versions inside the [patching](./patching.md#4-set-environment-variables) in step 2.
- If this is a **patch release**, update the versions inside the [patching](./patching.md#4-set-environment-variables) in step 4.

Create a new PR against the base branch:

```bash
git checkout -b bumping_${NEXT_VER}
git commit -a -m "chore(version): bumping version to ${NEXT_VER}"
git push origin HEAD
gh pr create --title "chore(version): bumping version to ${NEXT_VER}" --body "Bumping version to ${NEXT_VER}" --base ${BASE_BRANCH}
```

Wait for the PR to be approved and merged into the main branch.

## 10. Update the Website

Create a new announcement post on the
[blog](https://pactus.org/blog/) and update the
[Road Map](https://pactus.org/about/roadmap/) and
[Download](https://pactus.org/download/) pages.
Additionally, update the new release on the
[GitHub Releases](https://github.com/pactus-project/pactus/releases) page.
If gRPC APIs has changed in this version,
be sure to update the [API documentation](https://docs.pactus.org/api/) and
[Python SDK](https://github.com/pactus-project/python-sdk) accordingly.

## 11. Celebrate 🎉

Before celebrating, ensure that the release has been tested and that all documentation is up to date.
Don't forget to update dependencies after major releases (see [Update Dependencies](./update-dependencies.md)).
