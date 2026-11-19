# Changelog

## v20.2.0-ais.1

First release of the AIS fork.

- Based on `nielsenko/realm-dart@upgrade` commit `804483aab9` (analyzer 12 /
  build 4 / source_gen 4 migration of upstream `realm_generator 20.2.0`).
- `analyzer` constraint widened to `">=12.1.0 <15.0.0"`.
- `PseudoType.getDisplayString` accepts `includePositionalParameterNames`, so
  the generator compiles on analyzer 14.4.0 and newer as well as older releases.
- Standalone package: `resolution: workspace` removed, `publish_to: none` added.
- Test harness no longer assumes a pub-workspace root.

For history before the fork, see the upstream changelog at
https://github.com/realm/realm-dart/blob/main/CHANGELOG.md.
