# Migration Checker

This github action checks if migration and source code changes committed in the same pull request.
If both are commited in a same pull request, action will report failure.

## Motivation

It is dangerous to commit DB migration SQL codes and source codes in a single pull request.
Sometimes it might be OK, but in some cases, DB migration will affect source code changes due to execution sequences.

For example, if DB migration executed first, then source code being deployed after, error will happen in the following case:
1. Drop index X in DB migration
2. Remove index X usage in source code

In this case, step 2 should be executed first.

It is hard for human being to be careful enough to judge the execution sequence, so I believe a solution for safer release is to NOT commit DB migration and source code changes in a single commit.
This restriction will remind the author to think of the execution sequence and the impact of the DB migration.

## How to use

For example, if you have DB migration SQL in `/db/migrations/AAA` and `/db/migrations/BBB` folder in your repository, you can use this action in your workflow like this:

```yaml
name: Migration Checker

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check-changes:
    runs-on: ubuntu-latest
    env:
      MIGRATOR_PATH: |
    steps:
      - name: Check migration and source code changes
        uses: nerocrux/migration-checker@main
        with:
          migration_code_paths: |
            db/migrations/AAA
            db/migrations/BBB
          ignore_paths: |
            .github/
```

- Multiple DB migration SQL directories can be configured in `migration_code_paths` (line delimiter).
- You can ignore certain paths with `ignore_paths` so their changes won't be count when DB migration exists.

