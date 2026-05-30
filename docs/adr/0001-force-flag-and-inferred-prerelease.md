# Force flag for re-triggering failed publish jobs; isPrerelease inferred from version string

The publish workflow originally used an `isPrerelease` boolean input and an `inputs.version` escape hatch that could carry a prerelease suffix — but silently ignored that suffix in favour of `PSData.Prerelease` from the manifest. This created a class of contradictions with no clear resolution rule, and left no clean way to re-trigger a failed `publish` job when `create_release` had already succeeded (re-running the full workflow would try to create a duplicate tag).

We replaced `isPrerelease` with inference: a prerelease suffix is present in the reconciled version string or it isn't. We added `inputs.force` to bypass the PSGallery existence check; the intended re-trigger pattern is `force: true, create_release: false, publish: true`. `inputs.version` is now a pure version-override escape hatch — a bare version (e.g. `1.2.3`) means stable release even if the manifest carries a prerelease suffix.

## Considered options

- **Re-run failed jobs only** — GitHub Actions supports re-running only failed jobs, but `create_release` and `publish` are independent jobs; if `create_release` succeeded and `publish` failed, a full re-run recreates the tag and fails. Per-job re-run doesn't help because `publish` depends on `check_version` outputs that aren't re-evaluated.
- **Separate dispatch workflow for publish-only** — cleaner separation of concerns, but adds another workflow file callers must know about; `force` achieves the same goal within the existing interface.
