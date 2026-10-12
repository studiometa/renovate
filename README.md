# Studio Meta's shared Renovate configuration

This repository holds the [Renovate](https://docs.renovatebot.com/) preset of the Studio Meta and ikko repositories. The preset is in [`default.json`](./default.json).

## Usage

Add `.github/renovate.json` to the repository:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>studiometa/renovate-config"]
}
```

Add only options that are specific to the repository, for example `baseBranches`. Do not copy rules of the preset into the repository: change the preset instead, so that all repositories get the change.

New repositories get a config file from the onboarding PR of Renovate (see [Onboarding](#onboarding)).

## What the preset does

### Library ranges

Most of our packages are libraries. Their dependency ranges are part of what we publish, so Renovate must not make them narrower:

- npm `dependencies` keep their range: Renovate updates the lock file only (`rangeStrategy: update-lockfile`). A new major replaces the range.
- composer `require` ranges are widened (`^1.0` becomes `^1.0 || ^2.0`).
- `peerDependencies` ranges are widened.
- `devDependencies` are pinned to exact versions, because they do not ship to users.
- `engines` and the `php` constraint are not updated: a higher minimum is a breaking change for consumers, and a decision of the project.
- The Node.js version of `.nvmrc` is updated on majors only.

### Labels and the changelog

Every Renovate PR has the `renovate` label. PRs that do not change what users install also have the `no-changelog` label, so the changelog check of [weareikko/changelog](https://github.com/weareikko/changelog) does not ask for a fragment.

PRs that update runtime dependencies (npm `dependencies`, `peerDependencies`, `optionalDependencies` and composer `require`) do not have `no-changelog`. These updates ship to users, so the release lists them under `### Dependencies`.

Security PRs also have the `security` label.

### Grouping and automerge

- Patch and minor updates of a dependency come in one PR.
- `devDependencies` (and composer `require-dev`) are grouped in one PR for each package directory.
- GitHub Actions updates are grouped in one PR.
- Renovate waits 3 days after a release before it proposes it.
- Patch updates, pins, GitHub Actions updates and lock file maintenance are automerged when all status checks pass, on Monday mornings before 4 AM (Europe/Paris). Renovate always opens a PR first.
- Lock file maintenance runs on Monday mornings before 6 AM.

### Major updates

Major updates need an approval on the Dependency Dashboard issue of the repository. Tick the update on the dashboard and Renovate opens the PR.

### Security

Renovate opens a PR for each vulnerability that GitHub (Dependabot alerts) or [OSV](https://osv.dev/) reports. Security PRs do not wait for the 3 days delay, the schedule or the dashboard approval.

For GitHub alerts, enable the Dependency graph and Dependabot alerts in the settings of the repository.

### GitHub Actions

Actions are pinned to a commit SHA, with the full version in a comment (`actions/checkout@<sha> # v4.1.4`). Renovate updates the SHA and the comment together.

## Mend Renovate app

The [Mend Renovate app](https://github.com/apps/renovate) runs this preset. It is installed on the `studiometa` and `weareikko` organizations, on all repositories.

- Install or configure it in the settings of the organization on GitHub (Third-party access, GitHub Apps, Renovate).
- See the jobs, the logs and the mode of each repository (silent or interactive) on the [Mend Developer Portal](https://developer.mend.io).

### Onboarding

The Mend app opens an onboarding PR (Configure Renovate) in each repository that has no Renovate config. The PR adds `renovate.json` at the root of the repository. A repository uses Renovate only after its onboarding PR is merged.

In the `studiometa` organization, the onboarding PR proposes `local>studiometa/renovate-config` automatically, because this repository is named `renovate-config` and has a `default.json` file. The Renovate docs ([Group/Organization level presets](https://docs.renovatebot.com/config-presets/#grouporganization-level-presets)) say: "Renovate will check for a repository called `renovate-config` with a `default.json` file in the parent user/group/org of the repository. [...] If found, that repository's preset will be suggested as the sole extended preset, and any existing `onboardingConfig` config will be ignored/overridden." On github.com, `local>studiometa/renovate-config` loads the same preset as `github>studiometa/renovate-config` ([Local presets](https://docs.renovatebot.com/config-presets/#local-presets)). Before you merge the onboarding PR, replace `local>` with `github>` to use the same form as [Usage](#usage).

The `weareikko` organization has no `renovate-config` repository, so its onboarding PRs propose the default config of the Mend app (`config:recommended`). Before you merge such a PR, replace its config with the configuration of [Usage](#usage).

## Previous self-hosted runner

This repository ran Renovate itself with a GitHub Actions workflow and the `RENOVATE_TOKEN` secret. The Mend app replaces it. An administrator of the repository can delete the `RENOVATE_TOKEN` secret.
