# Branch Manager Handbook

- [Branch Manager Handbook](#branch-manager-handbook)
- [References](#references)
- [Overview](#overview)
  - [Minimum Skills](#minimum-skills)
- [Prerequisite](#prerequisite)
- [Safety Check](#safety-check)
  - [First Alpha Mock Build](#first-alpha-mock-build)
  - [Alpha Mock Release](#alpha-mock-release)
- [Build and Release](#build-and-release)
  - [Alpha Build](#alpha-build)
  - [Branch Creation](#branch-creation)
  - [Beta Build](#beta-build)
  - [RC Build](#rc-build)
  - [Official Build](#official-build)
  - [Debs and RPMs](#debs-and-rpms)
  - [Release](#release)
  - [Release Validation](#release-validation)
- [Branch Fast Forward](#branch-fast-forward)
  - [Code Freeze, Thaw](#code-freeze-thaw)
  - [Reverts](#reverts)
- [Cherry Picks](#cherry-picks)
- [Staging Repositories](#staging-repositories)

---

# References
* [Old old anago doc](https://docs.google.com/document/d/1Qoqz5IZYBp6A-Q_R9CGhMAc358ykOiE49GXZU9r5usQ/edit#heading=h.s71iha1627td)
* [Summary of discussion with Caleb about move from anago to gcb](https://groups.google.com/d/topic/kubernetes-milestone-burndown/YdHa51d95VI/discussion)
* [Non-release-workflow-specific, generic gcbmgr documentation](https://github.com/kubernetes/release/blob/master/README.md)
* [Patch release notes from exercising the new gcbmgr flow](https://docs.google.com/document/d/1x-GQDZpKk3WajtSnO0axDazE9Xs2mOSVgjziIuTWNO0/edit)

# Overview
The release branch manager is responsible for cutting a version of [Kubernetes](https://github.com/kubernetes/kubernetes/releases). Each release is a three month cycle where as branch manager:
1. You will cut releases of v1.X as specified on the Timeline in `sig-release/releases/release-1.X/README.md`
1. Participate in weekly release team meetings
1. Fast-forward the release branch on a daily basis as soon as the release branch `release-1.X` [has been created](#branch-creation).
1. Commit about approximate 8 hours or less in a week of your time
1. You will also get to select shadows and guide them in preparation to become the section lead for the next cycle.

For example, as branch manager of v1.15, the release cut dates as specified in the Timeline can be found at [`sig-release/releases/release-1.15/README.md`](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.15). The bulk of your time commitment is during release cut days.

## Minimum Skills
* Familiarity with basic Unix commands and able to debug shell scripts.
* General knowledge of Google Cloud (Cloud Build and Cloud Storage).
* Open to seeking help and can communicating clearly.

# Prerequisite
* Contact [Kubernetes Build Admins][kubernetes-build-admins], identifying yourself as the incoming release branch manager for the current release team and requesting to be added to the special privilege group for the GCP project used to build the releases
   * TODO: should the release lead also request some type of read access here for themselves?
   * TODO: should the branch manager shadows also get at least read access at the beginning?  If a shadow is going to exercise the process on behalf of the lead, they will need full read/write access ahead of time.
   * TODO: can the group name be identified here in a clear/transparent way?
* Ask the list owner(s) to give you access to **post** to these mailing lists:
   * [kubernetes-announce][k-announce-list] via owner contact form [here][k-announce-request]
   * If you haven't received access after 24 hrs, contact the [Release Team][kubernetes-release-team].
* Development client machine setup:
   * Linux.  MacOS/Windows are not supported by the scripting today.  However, running the [release tooling] inside a Linux image on MacOS' docker engine works well.
   * ssh configuration set up for GitHub (either static .ssh/config or ssh agent works)
   * run `gcloud init` using the identity that you had authorized in the prior bullet point
   * git clone and checkout the [k/release repo](https://github.com/kubernetes/release) for the `gcbmgr` and `release-notify` scripts
   * install [Google Cloud SDK](https://cloud.google.com/sdk/install)
   * ability to run `sendmail` from local Unix command line (BUG: should release-notify script be sending from GCB instead of the local machine, and can it?)
   * branchff: requires ability for your user to write /usr/local/ directory (BUG),
     sudo priv's, and membership in https://github.com/orgs/kubernetes/teams/kubernetes-release-managers
* Join these mailing lists to keep updated:
   * [kubernetes-sig-release](https://groups.google.com/forum/#!forum/kubernetes-sig-release)
   * [kubernetes-release-team](https://groups.google.com/forum/#!forum/kubernetes-release-team)

[k-announce-list]: https://groups.google.com/forum/#!forum/kubernetes-announce
[k-announce-request]: https://groups.google.com/forum/#!contactowner/kubernetes-announce
[release tooling]: https://github.com/kubernetes/release

# Safety Check
Run the following command to affirm your release repo checkout is in a good state, your development client is fully configured, and your user identity is authorized for the builds:
* `./gcbmgr`
   * should run successfully and even show some green colored "OK" words
* `./gcbmgr list`
   * should see for example: 1.9, 1.10, 1.11 builds

## First Alpha Mock Build
Start staging a build by running:
* `./gcbmgr stage master`
   * returns relatively quick
   * there`s a link you can click to pull up a browser and see build status tail
   * or `gcbmgr tail` to watch
* `./gcbmgr list`
   * You should now see your new job running in addition to those shown above in the “safety check” list command invocation.
   * Scan the output for failures or warnings.

Probably the build here fails, especially early in the release cycle.  By default the `gcbmgr stage master` command is running and automatically looking for a place where [release blocking test results](https://k8s-testgrid.appspot.com/sig-release-master-blocking) were all green, which traditionally has not happened in Kubernetes on an ongoing basis.  WE REALLY WANT (and need) TO GET THERE.  Quality needs to be a continual focus.  But in the meantime, acknowledging today especially for an early alpha or beta release, it is possible to just build...
* `./gcbmgr stage master --build-at-head`
   * Rather than having GCB find a point in the git history that had clean CI signal and building automatically from that point, we instead indicate we want to build explicitly from the git head.
   * This takes some time (approximately 1 hour is the current norm).  It’s building all the bits for a bunch of target operating systems and hardware architectures.
* `./gcbmgr tail`
   * You can watch the build progress.
   * Scan the output for failures and warnings.
   * For a successful build information is emitted on your next steps, including a build version identifier for the staged build, for example “v1.12.0-alpha.0.2622+7ac32a4f7a7ecf”.  This string is passed on to the release scripting, which is nicely emitted for you ready to copy and paste.
   * TODO: what guidance can we give on typical failures and triaging?

## Alpha Mock Release
After building comes the actual releasing, but this may be intentionally delayed.  For example the branch manager may stage a build from the head of the release branch late in the release cycle, doing so in the morning so that it is fully build and would releasable in the afternoon, pending CI signal results from that branch head.  If the results are good and the release team gives the go ahead, within minutes the release is published.  If the build were to only happen after the receipt of clean CI results, that could give additional hours of lag.  This of course presumes reproducible builds and that CI builds and tests are meaningful relative to the release builds.  There is always a risk that these diverge, and this needs managed broadly by the project and the release team.

* `./gcbmgr release  master --buildversion=v1.12.0-alpha.0.2622+7ac32a4f7a7ecf`
   * This copies the staged build to public GCP buckets at well-known urls for the truncated release version string.  The unique build staging identifier will subsequently be just “v1.12.0-alpha.1”, even though the staged build had an “alpha.0” in its name
   * NOTE: This is confusing.  The v1.12.0-alpha.0 tag was created automatically in the past when the v1.11 branch was forked, but it wasn’t actually build.
   * NOTE: This only just did a “mock” release.  Did everything indicate success?  Ok then…
* `./gcbmgr release master --buildversion=v1.12.0-alpha.0.2622+7ac32a4f7a7ecf --nomock`
   * NOTE: this fails too.  Mock builds can only be mock released.  A nomock release requires a nomock build to be staged.  But don’t fret, because all this mock building affirmed things work for your build client machine and userid, which is good!

# Build and Release
## Alpha Build
With all of that background success and knowing it’s probably a very bad idea to just skip that and come straight here, stage a releasable build by running:
* `./gcbmgr stage master --build-at-head --nomock`

Builds against master are implicitly a next alpha.  The gcbmgr/anago scripting automatically finds and increments the current build number.

## Branch Creation
The first time a specific branch is mentioned for the release instead of "master", for example:
* `gcbmgr stage release-1.12 --build-at-head --nomock`

Then behind the scenes anago will do a branch create and push.
`prow`’s `branchprotector` [runs once per day](https://github.com/kubernetes/test-infra/blob/a362b01ca57be7c170f7ef51c2f14dfd98986669/prow/cluster/branchprotector_cronjob.yaml#L1-L6) and automatically adds branch protection to any new branch in the `k/k` repo.

New release branch creation (for example: release-1.12) automatically also triggers an alpha.0 build for the subsequent release (for example: release-1.13). This means that the staging step will take about twice as long, as it will stage both versions `v1.12.0-beta.0` and `v1.13.0-alpha.0`. The release step will also take a bit (maybe a couple of minutes) longer, but not significantly.

Once the branch is created:
- Insert the patch release manager into the branch manager list in the prow configuration ([example](https://github.com/kubernetes/test-infra/blob/a362b01ca57be7c170f7ef51c2f14dfd98986669/prow/plugins.yaml#L151-L162)).
- Notify the [test-infra team](../test-infra), so they can setup the jobs & testgrid for the new branch.

## Beta Build
Builds against a release-1.X branch are implicitly a next beta.  The gcbmgr/anago scripting automatically finds and increments the current build number.
* `./gcbmgr stage release-1.12 --build-at-head --nomock`

As the first build from the new release branch, publication this build does not send out the typical changelog detail notification, but rather will only send a shorter message with subject line "k8s release-1.12 branch has been created".

## RC Build
Adding the --rc flag switches behavior on to building release candidates.  Again the gcbmgr/anago scripting automatically finds and increments the current build number.
* `./gcbmgr stage release-1.12 --rc --build-at-head --nomock`

In an perfect world, `rc.1` and the official release are the same commit. To
get as close to that perfect state as we can, the following things should be
considered:

1. All the PRs in the milestone should merge to master

   [`is:pr is:open milestone:v1.14`][pr-milestone-query] should be empty, there
   should be no open PR for that milestone.

   This means you (the release team) should really push on PRs towards the end
   of code freeze or assess if they can be removed from the milestone and can
   be punted to a next release.

2. `rc.1` should be staged and released

   Make sure that all the changes that have been merged in master also made it
   into the release branch. You can also do a
   [`branchff`](#branch-fast-forward) to see the state of the two branches and
   potentially pull any remaining PR from master into the release branch.

   At this point in time, the master and the release branch are the same.
   Nothing new gets merged into master, because we are still in code freeze.
   Therefore, it is safe to cut the release candidate.

3. Code Freeze can be lifted

   Technically we can keep code freeze in place after `rc.1` was cut. However,
   we should aim at lifting code freeze relatively quickly after `rc.1`.

   Otherwise we might have a mix of PRs against master, some have been merged
   in code freeze and for the milestone, just after `rc.1`, and others have
   been merged when code freeze has been lifted. We might miss this specific PR
   in the plethora of PRs that [tide] merges after code thaw, and we might
   miss that this PR actually needs to be cherry-picked into the release
   branch.

[pr-milestone-query]: https://github.com/kubernetes/kubernetes/pulls?utf8=%E2%9C%93&q=is%3Apr+is%3Aopen+milestone%3Av1.14
[tide]: https://git.k8s.io/test-infra/prow/tide

## Official Build
* `./gcbmgr stage release-1.12 --official --build-at-head --nomock`

In addition to `v1.12.n` this will also build and stage the subsequent patch's
`beta.0`, in this example `v1.12.(n+1)-beta.0`. Similar to [creating a new
branch](#branch-creation), the staging step will take about twice as long, the
release step will also take a couple of minutes more.

## Debs and RPMs
These require an additional layer of build and publish, which is currently still done by a Googler.  After building the official release, but before release notification, contact [Kubernetes Build Admins][kubernetes-build-admins] indicating for which build (for example: 1.12.0) Debs and RPMs are required.

These build relatively quickly and should be available ahead of sending the release notification, especially for the official release when the worldwide community will attempt to get the new artifacts.  Since they build from the release tag, the first release step below is a pre-requisite for initiating the package builds.

TODO: How do we automate this? These should not just be made but also validated prior to the release-notify phase. See issue: [kubernetes/release/#631](https://github.com/kubernetes/release/issues/631)

## Release
Releasing has multiple phases.  In the first phase the non-public build artifacts are published.  In the second phase the email notification goes out to the community.
* `./gcbmgr release master --buildversion=${{VERSIONID}} --nomock`
* ...interface with Google staffing to complete Debs and RPMs build, as per the prior section...
* `./release-notify v1.12.0-alpha.1`
* `./release-notify v1.12.0-alpha.1 --nomock --mailto=${{YOUREMAILADDRESS}}[a][b]`

## Release Validation
* The build tag and release artifacts become [visible on GitHub at https://github.com/kubernetes/kubernetes/releases](https://github.com/kubernetes/kubernetes/releases)
* The release is logged automatically by [k8s-release-robot](https://github.com/k8s-release-robot) in [sig-release](https://github.com/k8s-release-robot/sig-release)
* CHANGELOG-1.XX.md automatically hits the k/k repo: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md)
* TODO: Is there any scripting which will scan the GCP buckets and confirm all expected artifacts were published?  It seems no?  Because periodically users report just after a release that some file is missing, as if a sub-copy had failed?  Would be nice to have build-published-ok.sh or some such, and even a periodic daily rescan of old builds also as some prior build artifacts have somehow gone missing and needed re-published.

# Branch Fast Forward
Run simply as:
* `./branchff release-1.12`

This is done daily.

Earlier in the release the exact time of running [`branchff`] might just be at the discretion of the branch manager, as agreed upon with the release lead.

Later in the release it will probably become more important to align with the release lead and the CI signal team (and probably other release team members).
The exact time for pulling in the changes from master to the release branch might depend on the features that have or are to be merged. Considerations could be:
- We should run [`branchff`] sooner, before `$bigFeature` so we have a signal in the release branch before that fetature was brought in
- We should run [`branchff`] later, after `$theOtherFeature` has been merged, so we get signal on that feature from both the master and the release branch


The first time the [`branchff`] tool is run on a branch it will
- do a clone of `k/k` to a temporary directory
- run a git merge-base
- run a few cleanup scripts to ensure the branch’s generated API bits are in order  
  (master branch content will move on toward version `n+1`, while release branch needs to stay at version `n`)
- commit the results of those scripts to the branch
- push to the GitHub remote release branch

Except again, we have a safety net and the default is a mock run.

The script encourages you before committing to:
```
Go look around in /usr/local/google/nobody/branchff-release-1.14/src/k8s.io/kubernetes to make sure things look ok:
# Any files left uncommitted?
* git status -s
# What does the commit look like?
* git show
# What are the changes we just pulled in from master?
#   will be available after push at:
#   https://github.com/kubernetes/kubernetes/compare/641856db18...6ac5d02668
* git log origin/release-1.14..HEAD

OK to push now? (y/n):
```

It is critically important to do this during code freeze.  Look through the git log for example with:
```
git log origin/release-1.12..HEAD
```

Each and every commit ought to be something the release team has visibility into.  Each merge commit indicates a PR number and owner.  Invest in researching these.  If unexpected code was merged to master, use your judgement on whether to escalate to the release team and SIG leadership associated with PR to question whether the commit is truly targeted for the release.

**The release team and the branch manager are the final safety guard on the release content.**

Once you know your environment is good and running for the fast forwards:
* `./branchff release-1.12 --nomock`

Subsequent runs will simply be merging in changes from master to the branch, keeping the previous API fixup commits on the branch.

[`branchff`]: https://git.k8s.io/release/branchff

## Code Freeze, Thaw
Code merge restriction periods have been implemented by a combination of prow plugins config, submit-queue config, and tide.  The Test Infra Lead, Branch Manager and Release Lead coordinate checking in whichever config changes are required to enable and disable merge restrictions.

## Reverts
During code freeze it is especially important to first look at the list of commits on master since the prior fast forward, scanning their content and issue/PR artifacts to ensure they are changes expected for this milestone.  There merge-blocking mechanisms are relatively weak.  It is possible still for some people to write directly to the repo (bypassing blocking mechanisms) as well as for well intentioned milestone maintainers to approve a merge incorrectly.  The branch manager is a last pair of eyes.

If code incorrectly hits master it should be reverted in master.  Alternatively, release branch management must go to all cherry picks, picking around the errant commit.

# Cherry Picks

Once code freeze is lifted, and for the post-release patch management process, commits are cherry picked from master.

The current documentation in the [contributor guide for cherry picks](https://git.k8s.io/community/contributors/devel/sig-release/cherry-picks.md) should be generally sufficient.  There are a couple prerequisites to running the script which are outlined in that guide.

The cherry pick script is also fairly self documenting in terms of how to invoke the command.

There has been quite a bit of recent discussion (see: [1](https://github.com/kubernetes/community/pull/2408), [2](https://github.com/kubernetes/community/pull/1980)) around improving both the cherry pick process process and its documentation.  Watch for and contribute to improvements.


After the official release has been published, the [patch release
team](#../patch-release-manager/) will take over in handling cherry picks.
In the time between code thaw and the official release, cherry picks are the
responsibility of the branch management team. Consider the following when
assessing the cherry-picks:

- Check regularly if there are new cherry picks with
  [`is:open is:pr base:release-1.14label:do-not-merge/cherry-pick-not-approved`][cherry-pick-query]
- Consider that each cherry-pick diverges the latest release candidate that has
  been cut from the bits to be released as the official release
- Engage with the cherry pick requesters: How important is that cherry-pick,
  can it be pushed to a later release (patch or even minor), ... ?
- Discuss (especially controversial) cherry-picks in #sig-release or at the
  burndown meeting if you are not sure
- If certain cherry-picks go in, does this mean we want another release
  candidate, more time for the release candidate to soak (e.g. over the
  weekend)?


[cherry-pick-query]: https://github.com/kubernetes/kubernetes/pulls?utf8=%E2%9C%93&q=is%3Aopen+is%3Apr+base%3Arelease-1.14+label%3Ado-not-merge%2Fcherry-pick-not-approved

# Staging Repositories

The [publishing-bot](https://github.com/kubernetes/publishing-bot) is responsible for publishing the code in
[staging](https://git.k8s.io/kubernetes/staging/src/k8s.io) to their own repositories.

The bot also syncs the Kubernetes version tags to the published repos, prefixed with `kubernetes-`.
For example, if you check out the `kubernetes-1.13.0` tag in client-go, the code you get is exactly the same
as if you check out the `v1.13.0` tag in Kubernetes, and change the directory to `staging/src/k8s.io/client-go`.

[client-go](https://github.com/kubernetes/client-go) follows [semver](http://semver.org/)
and has its own release process. This release process and the publishing-bot are maintained
by SIG API Machinery. In case of any questions related to the published staging repos,
please ask someone listed in the following [OWNERS](https://git.k8s.io/publishing-bot/OWNERS) file.

The bot runs every four hours, so it might take sometime for a new tag to appear on a published repository.
The client-go major release (eg `v1.13.0`) is released manually a day after the main Kubernetes release.

[kubernetes-build-admins]: https://groups.google.com/forum/#!forum/kubernetes-build-admins
[kubernetes-release-team]: https://groups.google.com/forum/#!forum/kubernetes-release-team
