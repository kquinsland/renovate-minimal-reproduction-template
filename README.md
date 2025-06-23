### How are you running Renovate?

Self-hosted Renovate

### If you're self-hosting Renovate, tell us which platform (GitHub, GitLab, etc) and which version of Renovate.

GitHub

### Please tell us more about your question or problem

I am trying out renovate on a sample repo.
I am using a docker-compose file that looks like this:

```yaml
services:
  renovate:
    image: renovate/renovate:41.5-full
    # Has RENOVATE_PLATFORM: github + the tokens
    env_file:
      - ./env/self-hosted.env

    environment:
      # Defaults to 'info'
      LOG_LEVEL: debug
      # My config file (see below) mounted into this container
      RENOVATE_CONFIG_FILE: /opt/renovate/config.yaml
    volumes:
      - ./vols/renovate/base:/tmp/renovate
      - ./vols/renovate/config:/opt/renovate/
      - ./vols/renovate/job-logs:/logs
      - ./vols/renovate/db:/db
```

And here's the config file:

```yaml
---
onboardingConfig:
    # Presets: https://docs.renovatebot.com/presets-default/
    extends:
        - config:recommended

# Not clear if this loops in pre-commit or if it just updates it?
pre-commit:
    enabled: true
# Automatically rebase
rebaseWhen: "behind-base-branch"

repositories:
    - internal-test-org/dummy-repo

# Raise limits a bit
prHourlyLimit: 10
```

And inside `internal-test-org/dummy-repo/.github/renovate.json` I have:


```json
// Note: renovate supports comments in JSON files
// Please configure your IDE to allow comments in JSON files
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": [
        "config:recommended"
    ],
    "configMigration": true,
    "dependencyDashboard": true
}
```


Now, the [docs](https://docs.renovatebot.com/configuration-options/) say that `json` > `json5` and that comments are supported inside json files:

> Renovate supports JSONC for .json files and any config files without file extension (e.g. .renovaterc). We also recommend you prefer using JSONC within a .json file to using a .json5 file if you want to add comments.

However, when I run renovate, I get the following error:

```shell
❯ docker compose up
<...>
renovate-1  |  WARN: Configuration Warning (repository=internal-test-org/dummy-repo)
renovate-1  |        "configError": {
renovate-1  |          "validationSource": ".github/renovate.json",
renovate-1  |          "validationError": "Invalid JSON (parsing failed)",
renovate-1  |          "validationMessage": "Syntax error: expecting String near \n    // Re",
renovate-1  |          "message": "config-validation",
renovate-1  |          "stack": "Error: config-validation\n    at checkForRepoConfigError (/usr/local/renovate/lib/workers/repository/init/merge.ts:173:17)\n    at mergeRenovateConfig (/usr/local/renovate/lib/workers/repository/init/merge.ts:213:3)\n    at getRepoConfig (/usr/local/renovate/lib/workers/repository/init/config.ts:14:12)\n    at initRepo (/usr/local/renovate/lib/workers/repository/init/index.ts:58:12)\n    at Object.renovateRepository (/usr/local/renovate/lib/workers/repository/index.ts:66:14)\n    at attributes.repository (/usr/local/renovate/lib/workers/global/index.ts:184:11)\n    at start (/usr/local/renovate/lib/workers/global/index.ts:169:7)\n    at /usr/local/renovate/lib/renovate.ts:19:22"
renovate-1  |        },
<...>
```

And there is an issue opened on the repo that states as much.

But here's the kicker: when i run `pre-commit` on the same file... no issues.

```shell
# inside internal-test-org/dummy-repo
❯ cat .pre-commit-config.yaml
repos:
    - repo: https://github.com/renovatebot/pre-commit-hooks
      rev: 41.5.0
      hooks:
          - id: renovate-config-validator
            args: [--strict]
❯ pre-commit run --all-files
<...>
[INFO] Initializing environment for https://github.com/renovatebot/pre-commit-hooks:renovate@41.5.0.
<...>
renovate-config-validator................................................Passed
```


So my question is: what am I doing wrong here? Is this a bug or a misunderstanding on my part?

### Logs (if relevant)

<details><summary>Logs</summary>

```

Replace this text with your logs, between the starting and ending triple backticks

```

</details>


-----

Put your link to the Renovate issue or Discussion here.

https://github.com/renovatebot/renovate/discussions/36670
