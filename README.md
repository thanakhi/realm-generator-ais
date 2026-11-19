# realm_generator (AIS fork)

Code generator that turns Realm data-model classes into `.realm.dart` files.
This is a trimmed, single-package fork of the generator from the discontinued
[Realm Dart SDK](https://github.com/realm/realm-dart), kept alive so Realm model
codegen keeps working on Dart 3.13+ / Flutter 3.47+.

Only the generator is forked. The Realm **runtime** (`realm` / `realm_dart` /
`realm_common` 20.2.0) is consumed unchanged from pub.dev.

## Why this exists

The final upstream release, `realm_generator 20.2.0`, pins `analyzer: ^7.3.0`
and `build: 2.4.2` (exact). Because `realm_dart` lists the generator in its
runtime `dependencies`, those pins make any workspace that depends on Realm
unresolvable once `flutter_test` requires `analyzer >=8` (Flutter 3.47).

## Provenance

| | |
|---|---|
| Original project | `realm/realm-dart`, package `packages/realm_generator`, release 20.2.0 |
| Fork base | `nielsenko/realm-dart`, branch `upgrade`, commit `804483aab9e4da5ee1877e727f09b1a87bb22a01` (2026-07-19) — migrates the generator to analyzer 12 / build 4 / source_gen 4 |
| This repo | the `packages/realm_generator` directory of that commit, plus the changes below |

Changes on top of the fork base:

1. `pubspec.yaml` — `analyzer: ^12.1.0` widened to `">=12.1.0 <15.0.0"`. The
   upstream `<13` cap guarded the workspace-wide resolution for `ejson_lint`'s
   analyzer plugin ([dart-lang/sdk#63538](https://github.com/dart-lang/sdk/issues/63538),
   an IDE plugin-host hang); the codegen path is unaffected.
2. `pubspec.yaml` — `resolution: workspace` removed (this repo is standalone) and
   `publish_to: none` added.
3. `lib/src/pseudo_type.dart` — `PseudoType.getDisplayString` also accepts
   `includePositionalParameterNames`, which analyzer 14.4.0 added to
   `DartType.getDisplayString`. Without it the generator does not compile on
   analyzer 14.4.0 or newer. This is the only change under `lib/`, and it does
   not affect generated output.
4. `test/test_util.dart` — the test harness locates `package_config.json` by
   walking up from the package directory instead of assuming a workspace root.
5. `analysis_options.yaml` — the `ejson_lint` plugin entry removed (that package
   is not part of this repo).
6. Files that were symlinks into the upstream monorepo root (`LICENSE`,
   `analysis_options.yaml`) are real files here; `README.md` and `CHANGELOG.md`
   are rewritten for this fork.

## Compatibility

- Generated output is byte-identical to upstream `realm_generator 20.2.0` except
  that the two header comment lines swap order (`// GENERATED CODE - DO NOT
  MODIFY BY HAND` now precedes `// dart format width=80`), a `source_gen` 4
  behaviour.
- Pairs with the hosted `realm` / `realm_dart` / `realm_common` 20.2.0 runtime:
  the `generateRealmObjects` builder factory and `build.yaml` are unchanged, and
  the fork base does not modify `realm_common`.
- Test suite (59 tests) passes on analyzer 12.1.0, 13.3.0, 14.1.0, 14.3.0,
  14.4.0 and 14.7.0 (Dart 3.13).

## Usage

Consume it through a `dependency_overrides` entry, pinned to a tag:

```yaml
dependency_overrides:
  realm_generator:
    git:
      url: https://github.com/thanakhi/realm-generator-ais.git
      ref: v20.2.0-ais.1
```

Then run codegen as usual:

```bash
dart run build_runner build
```

Packages that resolve on their own (for example an `example/` app with its own
`pubspec.lock`) need the same override in their own `pubspec.yaml`.

## Development

Requires Dart 3.11 or newer.

```bash
dart pub get
```

```bash
dart test
```

The golden files live under `test/good_test_data/`.

### When a new analyzer release breaks the build

`PseudoType` implements analyzer's `DartType` interface, so an analyzer minor
release that adds or widens a `DartType` member can stop the generator from
compiling (this happened with 14.4.0). Run the suite against the new version
before relying on it.

To test a specific analyzer version, change the `analyzer` **constraint** in
`pubspec.yaml` to that exact version, delete `pubspec.lock` and run
`dart pub get` so pub picks matching `build_runner` / `source_gen` versions.
Do not use `dependency_overrides` for this: it forces the analyzer version
without re-solving the rest, and the resulting failures come from the other
packages rather than from this one.

## Syncing with upstream

`nielsenko/realm-dart@upgrade` is rebased from time to time. A later revision
(`b779f23`, 2026-08-31) rewrites generator internals and also changes
`realm_common`, which this fork deliberately does not track: the hosted
`realm_common 20.2.0` is immutable, so adopting that revision needs a fresh
evaluation (test suite plus output parity against a consumer) first.

## License

Apache License 2.0 — see [LICENSE](LICENSE). Original work copyright MongoDB,
Inc.; source files keep their upstream copyright and SPDX headers.
