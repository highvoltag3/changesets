# Decisions

This file is a discussion of some of the rules and design decisions that have gone into making changesets. The goal of all of these has been to make the experience of using changesets easy, while still providing the maximum value possible.

## How changesets are combined

Changesets are designed to be as easy to accumulate as possible. As such, when changesets are consumed, we flatten the version bumps into one single bump at the highest semver range specified.

For example: if you run `consume`, and a we have `packageA` at `1.1.1`, which has two `minor` changesets, and one `patch` changeset, we will bump `packageA` to `1.2.1`.

This allows changesets to be added and accumulated safely, with the knowledged that packages will only be released once at an appropriate version for the combined set of changesets, while still ensuring each change is captured in the changelog, with an indication of what kind of change it is.

## How dependencies are bumped

> NOTE: This refers specifically to a feature of changesets used in mono-repos

When changesets are generated, we check to see if the selected packages will leave semver for any other packages within the mono-repo.

For example, if I have two packages:

`packageA` at `1.1.1`

and `packageB` at `1.1.0` that depends on `packageA` at `^1.1.0`.

If I add packageA to a changeset with a `major` change, the version of `packageB` within the mono-repo should also be updated. If it is not, either `packageB` in the mono-repo will not use `packageA` in development, or `packageB` in development will not match an installation of `packageB` in prdouction.

As such we end up with a changeset that includes `pacakgeA` as `major` and `packageB` as `patch`.

All updating of dependencies is done as a patch bump. If you want to indicate a more significant change to `packageB` from consuming a new version of `packageA`, we recommend adding a second changeset specifically for `packageB`.

## Why we write files to disc

There are two reasons we chose to do this. The first is so the changeset descriptions are editable after creation, and a user can go in and change this as they desire. The second is that it means we are unoppinionated about your git workflows, with sqashing and modifying commits being completely safe, without fear of breaking a release.

## What distinguishes this from Semantic Release

If you have been looking at automating versioning previously, you may have come across [semantic release](https://github.com/semantic-release/semantic-release), or its mono-repo equivalent [lerna semantic release](https://github.com/atlassian/lerna-semantic-release). It's good to understand how changesets operates differently.

1. Changesets are designed for mono-repos first

This means we manage dependencies within the repository, which other tools do not do.

2. We commit our change information to the file system, instead of storing it in git.

See the above section on why we write files to disc

3. We use semver for specifying the change

When selecting the kind of change your package is, we do not specify any change types beyond `major`, `minor`, or `patch`. Semantic release allows you to specify a range of fields (bug-fix, feature) that it converts to an appropriate semver type. This is a design decision on our part to push adding this information into the changeset description itself.

## The versioning of peer dependencies

Currently, if you list a package as a peerDependency of another package, this causes the package with the peerDependency to
be released as a `major` change. This is because peerDependency changes will not be caught by package installation.

This decision is open for discussion.
