# Ocean build notes (fossify-notes)

This branch is a pinned build branch for the `fossify_notes` mobile-world
ledger. It is cut from upstream release tag **1.7.0** and currently contains no
code changes: re-verification on 2026-09-23 (Android 14 / SM-S9210) showed the
four open ledger items are either fixture/materialization gaps or helper-side
defects in `seed_android_env`, not defects in the Notes app at this tag.

- Upstream: https://github.com/FossifyOrg/Notes tag `1.7.0` (commit
  `786fc41f68d1aed8d82144a6de0127ed4dbe8b61`)
- License: GPL-3.0-only (unchanged; see LICENSE)
- applicationId: `org.fossify.notes`, versionName 1.7.0, versionCode 13

## Reproducible build

```sh
./gradlew :app:assembleFossRelease
```

On hosts without direct access to Maven Central / Google Maven / the Gradle
Plugin Portal, the build used host-local mirror configuration (not committed to
this fork): an init script under `~/.gradle/init.d/` pointing
`dependencyResolutionManagement` / `pluginManagement` repositories at
`https://mirrors.cloud.tencent.com/nexus/repository/maven-public` (which hosts
`org.fossify:commons:6.1.0`) and the Aliyun `gradle-plugin/public/google/
jitpack` mirrors.

The release artifact is unsigned (`app/build/outputs/apk/foss/release/
notes-13-foss-release-unsigned.apk`). The materialized APK was zipaligned and
signed with an Android debug key so it installs on stock devices without a
release keystore:

```sh
zipalign -p -f 4 notes-13-foss-release-unsigned.apk aligned.apk
apksigner sign --ks ~/.android/debug.keystore --ks-pass pass:android \
  --key-pass pass:android --ks-key-alias androiddebugkey \
  --out notes-1.7.0-ocean1-foss-release.apk aligned.apk
```

## Verified behavior (2026-09-23, SM-S9210, Android 14)

- Fresh GUI create/save/reopen of a multilingual plain-text note persists
  across HOME autosave and `am force-stop` + relaunch; the UI readback matches
  the authoritative GUI JSON export
  (`Settings → Migrating → Export notes`).
- `pm clear` twice restores the zero-note baseline; reset/re-init does not
  duplicate notes.

Operational note: autosave runs in `TextFragment.onPause()` / page-switch, so
workers must leave the editor (HOME / open another app / leave the page) before
force-stop for the last edit to be committed. BACK inside the editor can raise
the app's delete-note confirmation dialog; it must not be treated as a save
gesture.
