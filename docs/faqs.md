---
title: FAQs
metaTitle: "Actionsflow Getting Started"
---

# Getting help

Actionsflow is under active development. If you need help, would like to
contribute, or simply want to talk about the project with like-minded
individuals, we have a number of open channels for communication.

- To report bugs, file feature requests, or submit trigger requests: use the [issue tracker on Github](https://github.com/actionsflow/actionsflow/issues).
- To contribute code or documentation changes: submit a [pull request on Github](https://github.com/actionsflow/actionsflow/pulls).
- To communicate/get help from other Actionsflow users: join [Actionsflow Slack](https://join.slack.com/t/actionsflow/shared_invite/zt-h5tmw9cn-GbZ4fzU_vc_qB~nnS_2Lvg).

# FAQs

## 1. How Actionsflow deduplicate data?

The Actionsflow's smart deduplication can help you to skip the data which has seen, the deduplicated job is handled by Actionsflow, not by the trigger. If a trigger's data need to deduplicate, then it will provide a [`getItemKey`](./reference/trigger-api.md#getitemkey) method, Actionsflow will cache the deduplication keys to [Github Actions cache](https://github.com/actions/cache), then, next time Actionsflow get the same data, it will skip that. For example, [`rss`](./triggers/rss.md) use `guid` or `link` as the deduplication key.

## 2. How do I set a scheduled/cron event?

To set a cron event on Actionsflow, see [`on.<trigger>.config.every`](./workflow.md#ontriggerconfigevery).

## 3. How do I debug trigger outputs?

For example, if you want to debug the RSS trigger outputs, you can use the [`toJSON` function provided by Github](https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions#tojson) like this:

```yaml
on:
  rss:
    url: https://hnrss.org/newest?points=300
jobs:
  print:
    name: Print
    runs-on: ubuntu-latest
    steps:
      - name: Print Outputs
        env:
          outputs: ${{ toJSON(on.rss.outputs) }}
        run: |
          echo outputs: $outputs
```

## 4. How do I debug?

If you want to debug a specific trigger, you can pass [`debug: true`](./workflow.md#ontriggerconfigdebug) to the trigger.

If some errors occur, maybe you want to debug the issues. To enable Github Actions step debug logging, you must set the following secret in the repository that contains the workflow: `ACTIONS_STEP_DEBUG` to true. If so, Actionsflow will set `logLevel: debug`, so you can debug the details. For more about debugging please see [Enabling debug logging](https://docs.github.com/en/free-pro-team@latest/actions/managing-workflow-runs/enabling-debug-logging).

## 5. How do I clean the cache?

For some reason, you may want to delete the Actionsflow's cache. You can do this by manually running this workflow in your repository's Actions tab.

## 6. How do I run a single workflow?

When you have multiple workflow files, you may want to disable some of them. To do that you can set [`on.<trigger>.config.active`](./workflow.md#ontriggerconfigactive) to `false`, or you can use the [`--include`](./reference/cli.md#build) or [`--exclude`](./reference/cli.md#build) CLI arguments. Here's an `--include` example: `npm run build -- -i rss.yml`, or glob `npm run build -- -i "rss*"`.

## 7. `argument list too long` Error

You may see this error [`OCI runtime exec failed: exec failed: container_linux.go:370: starting container process caused: argument list too long: unknown`](https://github.com/actionsflow/actionsflow/issues/4) when running `act`. This is because your built workflow file is too large for [`act`](https://github.com/nektos/act) to handle. You can work around this by reducing your outputs using [`on.<trigger>.config.filterOutputs`](./workflow.md#ontriggerconfigfilteroutputs). For example:

```yaml
on:
  rss:
    url:
      - https://hnrss.org/newest?points=500
      - https://hnrss.org/show?points=100
    config:
      filterOutputs:
        title: 1
        link: 1
```

With this configuration, only `title` and `link` will be sent to the output workflow.
