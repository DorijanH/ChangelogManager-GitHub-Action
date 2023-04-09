<h1 align="center">
  <img width="128" height="128" src="docs/icon.svg" alt="logo" />

  Changelog Manager Github Action
</h1>

<div align="center">
  <p>Compiles the <i>change</i> files into CHANGELOG.md</p>
  <div>

  [![Test](https://github.com/Enterwell/ChangelogManager-GitHub-Action/actions/workflows/CI.yml/badge.svg)](https://github.com/Enterwell/ChangelogManager-GitHub-Action/actions/workflows/CI.yml)
  [![CodeQL](https://github.com/Enterwell/ChangelogManager-GitHub-Action/actions/workflows/codeql-anaysis.yml/badge.svg)](https://github.com/Enterwell/ChangelogManager-GitHub-Action/actions/workflows/codeql-anaysis.yml)
  [![GitHub issues](https://img.shields.io/github/issues/Enterwell/ChangelogManager-GitHub-Action?color=0088ff)](https://github.com/Enterwell/ChangelogManager-GitHub-Action/issues)

  </div>
</div>

## 🌱 Introduction
This is the *GitHub Action* version of an [Azure DevOps task](https://github.com/Enterwell/ChangelogManager/tree/main/Enterwell.CI.Changelog.DevOpsExtension) developed for the convenience of managing a [changelog](https://keepachangelog.com/en/1.1.0/), indirectly, using special *change* files.

#### What are the *change* files?

*Change* files are just files located in the ***changes*** directory with the following naming scheme:

```
<change_type> [<change_category>] <change_description>
```

Acceptable entries for the `<change_type>` are:

+ Added
+ Changed
+ Deprecated
+ Removed
+ Fixed
+ Security

This decision was inspired by following the [principles](https://keepachangelog.com/en/1.0.0/#how) for keeping a good changelog.

To avoid incorrect file naming and to ease this file creation process on the developer, [Visual Studio extension](https://github.com/Enterwell/ChangelogManager/tree/main/Enterwell.CI.Changelog.VSIX) and the [CLI helper](https://github.com/Enterwell/ChangelogManager/tree/main/Enterwell.CI.Changelog.CLI) were made.

This *Action* internally calls our [Changelog Manager tool](https://github.com/Enterwell/ChangelogManager/tree/main/Enterwell.CI.Changelog) by forwarding action inputs to it, which then inserts the appropriate section to the `CHANGELOG.md` and deletes all the contents of the ***changes*** directory.
 
We **highly** recommend that you read up on how and what exactly is it doing behind the scenes, as well as, learn how to use the `.changelog.json` configuration file to customize the tool's behaviour.

## 📖 Table of contents
+ 🌱 [Introduction](#-introduction)
+ 🛠️ [Prerequisities](#-prerequisities)
+ 🔖 [Versions](#-versions)
+ 🏗 [Development](#-development)
+ ☎️ [Support](#-support)
+ 🪪 [License](#-license)

## 🛠 Prerequisities

The following prerequisities need to be fulfilled in order for the action to work properly:

#### Changes directory

Automation agent where the action executes needs to have a ***changes*** directory containing the [*change*](#what-are-the-change-files) files created manually or with one of our helpers. If the directory does not exist, action will throw an error.

#### Changelog file

The action also needs to be able to find `CHANGELOG.md` file (naming is case-insensitive), otherwise, it will throw and stop executing.

*Important note*. As it currently stands, the action is inserting the newly compiled section **above** the latest changelog entry. So, the `CHANGELOG.md` needs to contain **atleast**

```
## [Unreleased]
```

## 🔖 Versions

This action currently has two versions: `v1` and `v2`.

### v1

The initial action originally developed to be ran ***only*** inside Windows runners.

#### 📝 Usage

#### Basic
```yaml
uses: Enterwell/ChangelogManager-GitHub-Action@v1
with:
  semantic-version: <major.minor.patch>
```

#### Advanced
```yaml
uses: Enterwell/ChangelogManager-GitHub-Action@v1
with:
  semantic-version: <major.minor.patch>
  changelog-location: ./
  changes-in-different-location: true
  changes-location: ./somewhere-else/changes
```

#### 📥 Inputs

#### `semantic-version`
**Required** A string (e.g. major.minor.patch) representing the new version of the repository.

#### `changelog-location`
**Optional** Location of the directory containing the `CHANGELOG.md` file.
  + Defaults to `./`

#### `changes-in-different-location`
**Optional** Boolean representing that the *changes* directory exists in a different location than the `CHANGELOG.md` file.
  + Defaults to `false`

#### `changes-location`
**Optional** Location of the `changes` directory.
  + If the previous input is set to `false`, changes location is set to `<location containing the CHANGELOG.md>\changes`.

#### 📤 Outputs

None

### v2

The smarter, new and improved action! 🎉

New features:
+ can be ran inside both, Ubuntu and Windows runners 🤖
+ removed explicit semantic version input
  + The task is smart enough to determine the next version by itself based on your current application's version and the changes you made! 🤯
+ can automatically (or explicitly) determine the project file and bump its version
  + this feature is **opt-in**
  + supported project types
    + NPM (`package.json`)
    + .NET SDK (`.csproj` with the `Version` (case-insensitive) tag)
+ outputs the newly bumped application's version as an output variable
+ ***only*** deleting [change](#what-are-the-change-files) files that were used to build the new changelog section and therefore eliminating the risk of accidentally deleting wrong files

#### 📝 Usage

#### Basic
```yaml
uses: Enterwell/ChangelogManager-GitHub-Action@v2
```

#### Advanced
```yaml
uses: Enterwell/ChangelogManager-GitHub-Action@v2
with:
  changelog-location: ./
  changes-in-different-location: true
  changes-location: ./somewhere-else/changes
  set-version-flag: true
  path-to-project-file: ./somewhere-else/[package.json | something.csproj]
```

#### 📥 Inputs

#### `changelog-location`
**Optional** Location of the directory containing the `CHANGELOG.md` file.
  + Defaults to `./`

#### `changes-in-different-location`
**Optional** Boolean representing that the *changes* directory exists in a different location than the `CHANGELOG.md` file.
  + Defaults to `false`

#### `changes-location`
**Optional** Location of the `changes` directory.
  + If the previous input is set to `false`, changes location is set to `<location containing the CHANGELOG.md>\changes`.

#### `set-version-flag`
**Optional** Boolean representing should the new version be set in the appropriate project file (`package.json` or `*.csproj`).
  + Defaults to `false`

#### `path-to-project-file`
**Optional** Path to the project file (`package.json` or `.csproj` with the `version` (case-insensitive) tag).
  + If the previous input is set to `true`, but this input is not passed in explicitly, the action will try to automatically determine the appropriate project file
  + If the previous input is set to `false`, this input is **ignored**

#### 📤 Outputs

#### `bumped-semantic-version`
Newly bumped semantic version based on the changes made.

#### Usage

```yaml
# github-workflow.yml
...
  steps:
    ...
    - uses: Enterwell/ChangelogManager-GitHub-Action@v2
      id: merge-changelog

    - run: echo ${{ steps.merge-changelog.outputs.bumped-semantic-version }}
    ...
...
```

## 🏗 Development

This action uses [pnpm](https://pnpm.io/) as its package manager, so in order to get quickly up and developing you will need to have it installed on your machine.

If you don't already have it, you can easily install it by using the following command (assuming you have [Node.js](https://nodejs.org/en) installed)

```bash
npm install --global pnpm
```

Now you can install dependencies using the following command

```bash
pnpm install
```

### Available commands

| Command | Description |
| ----------- | ----------- |
| `pnpm build` | Compile the TypeScript source code into JavaScript |
| `pnpm lint`  | Lint the source code |
| `pnpm test` | Test the action |
| `pnpm package` | Packages the action source code and its dependencies into `dist` folder |

## ☎ Support

If you are having problems, please let us know by [raising a new issue](https://github.com/Enterwell/ChangelogManager-GitHub-Action/issues/new).

## 🪪 License

This project is licensed with the [MIT License](LICENSE).
