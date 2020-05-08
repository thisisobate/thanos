---
title: 'Building A Versioning Plugin For Thanos'
type: proposal
menu: proposals
status: Approved
owner: thisisobate
Date: May 2020
---


## Problem

The Thanos website contains the docs area which is built by rendering the markdown files from the tip of master of the Thanos repository. This frequently causes confusion for users, as different Thanos versions have different features.

In this proposal, we want to solve this. In particular, we want to:

1. Build a Version Picker that serves as a tool which will aid easy access to other versions of the documentation. This picker will Include links to change version (the version must be in the URL).
2. A solid and robust Documentation structure.
3. Design a workflow for managing docs that integrates with Thanos Git workflow ie updating corresponding docs on pull requests, cherry-picks, etc.

## Motivation

Many Users (mostly developers) would often want to look through the docs of previous releases, but searching for it manually by looking through the entire GitHub repository is time-consuming and ineffective. Because the site is built by rendering the markdown files from the tip of master of the Thanos repository, it causes confusion for users, as breaking changes are often merged into master. The verioning plugin allows the user to access the docs of both latest and previous releases on the Thanos documentation page, and it also allows the developer to fetch, write, and fix the docs of both latest and previous releases.


## Requirements

#### User Story (Latest)

* As a Thanos developer, I want to be able to just write docs for my current `master` version
* As a Thanos developer, I want to build website within versioned docs with a single action
* As a Thanos developer, I don't want to store in a single repository version duplicated docs

#### User Story (Previous release)

* As a Thanos developer, I want to be able to fix docs for certain previous release.
* As a Thanos user, I want to be able to read docs for older version of Thanos

#### Use Case

##### Users
Thanos Developers, Hugo developers, and General Users

##### Precondition
The user visits the Thanos Documentation Page

##### Basic Course Of Events

1. The User indicates that the site is to show a list of docs (both latest and previous releases) by clicking the version picker (dropdown)
2. The site responds by displaying a list of versions allowing the user to make a selection.
3. User makes selection and the required docs renders on the page/site.

##### Postcondition

The site now has a version picker containing a list of versioned docs consisting of both latest and older versions.

## Proposed Solution

#### 1. Version Picker

Currently, the documentation is residing under the docs/ folder of Thanos repository. It is built by Hugo. It will have a proper drop-down menu just like prometheus [`drop-down menu`](https://prometheus.io/docs/introduction/overview/) which will enable proper versioning. This user facing tool will be built using HTML and CSS

#### 2. Documentation Structure

We want to propose a method called "Directory Sub Branching".
Directory Sub branching means creating different sub branches in the `versioned` folder of the Thanos repository. Since the current architecture of the Thanos website is this:

```|- website
    |- archetypes
    |- layout
    |- data
    |- static
    |- resources
    |- tmp
        |- public
        |- docs-pre-processed
```
We want to add aditional `versioned` folder within the website's `tmp` directory. For example:

```|- website
    |- archetypes
    |- layout
    |- data
    |- static
    |- resources
    |- tmp
        |- public
        |- docs-pre-processed
        |- versioned
            |- master
            |- version 0.13.0
            |- version 0.12.2
            |- other-folder-for-other-releases
```
_NOTE: `tmp` directory is not committed, just temporarily built. The current version of docs lives in the `master` folder_


#### 3. Building a versioning plugin

Creating a plugin that can automate these processes would save us alot of development time and stress. This approach promises to be very useful when it comes to versioning different release in the Thanos website.


##### Workflow

1. Developer makes all the necessary edit on `/docs` on master
2. Developer proceeds by committing a new Release (i.e Release 0.x)
3. CI run some `make web` or `Make web-serve`
4. Before anything, CI generate those docs and stores in a `versioned-tmp` folder
5. rest of web command is done

_NOTE: generated docs are not committed, just temporarily built._

## FAQ

This section consists of some important questions and answers that are frequently asked by users.

##### How to specify what part of docs you want to version without cloning every part of the docs? docs.yaml

We hope to have a single flexible configuration file (`docs.yaml`) that will help the developer in specifying what part of docs he/she needs without cloning all the docs. We could have this as a single file on master. The config file will look like this:

```versioned:
   default:
   - docs/components/*.md
   - docs/storage.md

  overrides:
     release-0.12:
     - docs/storage2221241.md
```
##### Alternative path

We could have this file on master, so the current `make web` will

1. Parse on master `docs.yaml` and decide.
1. Go to that particular release (`i.e release-0.10`)
1. Is there docs.yaml?
    * Yes - parse and decide
    * No - use docs.yaml from master

The design of docs.yaml will look like this:

```versioned:
   - docs/components/*.md
   - docs/storage2221241.md
```
##### How would the plugin look like?

CLI (`make generate-versioned-docs`)

##### Workplan in terms of plugin placement?

We hope to start in Thanos project, then move project outside to a separate repo.

##### How we do fixes to release docs for older release e.g v0.12?

We will edit the particular release (release-0.12) and commit. Then the tool fetches the latest on this branch and use it to generate docs. We expect to fetch docs for minor releases without breaking them with patches. We encourage immutability across all our release tags in Thanos.

##### How the tool discovers the releases for fixes?

By using regex. So instead of developer manually checking out to individual release branch, the tool handles it for them by using a regex to select and clone valid release branches. The tool (config file) will have a `release-branch-regex` field, and for Thanos, the regex structure would be something like this `release-(.*)`

**Please note that these approach has their pros and cons as well, of which we need to evaluate together. I will keep updating this document as I get more findings and possibly, we can reach to a conclusion on which approach we need to take before we commence implementation.**
