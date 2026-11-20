# Fork maintenance

This repository follows `chen08209/FlClash` and publishes its own Android
arm64 build. `sync-upstream.yaml` merges upstream `main` daily, tags the fork
commit with the upstream release tag, and dispatches the release build.

What this fork owns, and why a merge does not pick it up:

- `.github/workflows/build.yaml` and `.github/release_template.md` are
  replaced wholesale: a single Android arm64 artifact, a short README-style
  release body, and a release that is published before its assets are
  uploaded. `.gitattributes` marks both `merge=ours`, so upstream changes to
  them are dropped rather than conflicted. A local merge needs the driver:

  ```bash
  git config merge.ours.driver true
  ```

  Because the merge drops upstream's version of that workflow, its toolchain
  pins (`FLUTTER_VERSION`, `GO_VERSION`, `NDK_VERSION`) and its build command
  are not updated for free: after a sync, compare them with upstream's
  `build.yaml` and carry over anything a release needs. Flutter in particular
  has to satisfy the `flutter:` constraint in `pubspec.yaml`, and upstream
  moves the packaging entry point (`dart tool/package.dart <platform>`).

- `test/tool/release_contract_test.dart` is also `merge=ours`: it packages
  every row of upstream's release matrix and expects the fifteen assets that
  matrix publishes, which a one-artifact fork does not ship. This side asserts
  the single APK the fork does publish, and skips the matrix group.

- `android/app/build.gradle.kts` sets `applicationId = "com.follow.clash.homo"`
  so this build installs beside the official app. The `namespace` stays
  `com.follow.clash`, which is where the Kotlin and the manifest live.
- `lib/common/constant.dart` points `repository` at this fork, which is what
  the in-app update check reads.
- `build_config.yaml` and `pubspec.yaml` are upstream's and merge normally.

Release tags here reuse the upstream tag name but point at the fork's merge
commit, so a tag in this repository never matches upstream's commit for the
same version. Never move or recreate a tag once the release exists.

The `pages` workflow is disabled in the repository settings: it deploys the
upstream project site and is gated on `github.repository ==
'chen08209/FlClash'`, so it only produced skipped runs here.

CI does not run `tool/changelog.dart verify`: a fork tag has no
`changelog.json` entry of its own, so `verify` would reject every tag the sync
creates. Release notes fall back to the last 30 commits when the renderer
cannot derive them, which is why `upload` tolerates that failure.