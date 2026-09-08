# unknown-deps-count-fixture

Fixture repo for **CORE-7341**: a revision's cached `dependency_count` under-reports the
total when the revision has unknown (unresolvable) dependencies.

## Contents

`fossa-deps.yml` declares 4 referenced npm dependencies:

| Dependency | Exists on npm? | Expected result in FOSSA |
| --- | --- | --- |
| `lodash@4.17.21` | yes | known dependency |
| `core7341-fake-dep-alpha@9.9.9` | no | unknown dependency |
| `core7341-fake-dep-beta@9.9.9` | no | unknown dependency |
| `core7341-fake-dep-gamma@9.9.9` | no | unknown dependency |

## Expected symptom (before the fix)

After analysis:

- Dependencies tab table total (live count): **4** (1 known + 3 unknown)
- Header stat / cached `Revisions.dependency_count`: **2** (1 known + 1 collapsed `NULL` lock row)

The cached count is written at `modules/DependencyManager/setDeepDependencies.ts` from
`postOverwriteLocks.length`, where all unreachable dependencies collapse into a single
`locator: 'NULL'` row. Note the wrong value self-heals if an issue scan runs after the
revision reaches steady state, so check the DB value right after the build finishes.
