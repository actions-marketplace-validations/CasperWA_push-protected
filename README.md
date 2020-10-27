# Push Protected - GitHub Action

_**Push to "status check"-protected branches.**_

Push commit(s) to a branch protected by required [status checks](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-status-checks) by creating a temporary branch, where status checks are run, before fast-forward merging it into the protected branch, finally removing the temporary branch.

> **Note**: Currently this action _only_ supports status checks that are GitHub Action status checks, i.e., no third-party status checks are currently supported (like, e.g., protecting a branch with Travis CI checks).
> This is expected, however, to be added in the future.

## Update your workflow

To successfully have the required status checks run on the temporary branch, you _may_ need to add it to the workflow(s) that is/are responsible for the required status checks.

If you are using `on: [push]` and not

```yml
on:
  push:
    branches:
```

or similar, i.e., if you are running the workflow(s) for _all_ kinds of push actions, **there is no need to update your workflow(s)**.

_However_, if you are filtering on which branch/tag names trigger your workflow(s), then keep reading.

In order to not have to continuously update the yml file(s), the temporary branches all have the same prefix: `push-action/`.
The complete name is `push-action/<RUN_ID>/<UUID>`, where `<RUN_ID>` is the unique GitHub Actions run ID for the current workflow run, and the `<UUID>` is generated using `uuidgen` from the `uuid-runtime` library.

Getting back to adding the temporary branch(es) to your workflow(s)'s yml file(s), it can be done like so:

```yml
on:
  push:
    branches:
      - 'push-action/**'
```

An example can also be seen in this action's own [test workflow](.github/workflows/test_status_checks.yml).

## Notes on `token`

If you are using this action to push to a GitHub [protected branch](https://help.github.com/en/github/administering-a-repository/about-protected-branches), you _need_ to pass a [personal access token (PAT)](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line), preferrably as a [secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets), to the `token` input.
This can be done as such:

```yml
name: Pushing to the protected branch 'protected'
uses: CasperWA/push-protected@v1
with:
  token: ${{ secrets.PUSH_TO_PROTECTED_BRANCH }}
  branch: protected
```

**Note**: If you are _not_ pushing to a protected branch, you can instead use the [`GITHUB_TOKEN`](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token) secret, which is auto-generated when you use GitHub Actions.
I.e., `token: ${{ secrets.GITHUB_TOKEN }}`.

The reason why you can not use the `GITHUB_TOKEN` secret when pushing to a branch that is protected by required status checks, is that using this as authentication does not trigger any webhook events, such as 'push', 'pull_request', etc.
This event trigger is a **MUST** for starting the required status checks on the temporary branch, which are necessary to run in order to be able to push the changes into the desired branch.

The PAT should have a scope appropriate to your repository:

- Private: _repo_
- Public: *public_repo*

It is recommended to not add unneccessary scopes to a PAT that are not needed for its intended purpose.

## Usage

This action is inspired by [`ad-m/github-push-action`](https://github.com/marketplace/actions/github-push) and to ease its use, it implements some of the same functionality.
The inputs `repository`, `branch`, and `token` are similar, where the `token` input has been renamed from `github_token`.
The `ad-m/github-push-action` input `directory` (and temporarily `force` and `tags`) are the only unsupported inputs.

To use this action, the workflow is also similar to `ad-m/github-push-action`.

First, you **MUST** use the [`actions/checkout`](https://github.com/marketplace/actions/checkout) action to checkout your local repository if you wish to make changes to it before pushing these changes to `branch`.

Second, you then make your changes.
Either as an explicit step of the workflow or you can run a separate script that incorporates all your changes.
If you wish to add tags or similar this can also be done now.

Finally, you run this action, specifying the inputs to your needs (see overview of all inputs below), and the action will take care to create a temporary branch with your changes, where the status checks will run, before merging it into your target branch (`branch` input), effectively "pushing" your local changes to the protected branch.

## Inputs

All input names in **bold** are _required_.

| Name | Description | Default |
|:---:|:---|:---:|
| **`token`** | Token for the repo.<br>Used for authentication and starting 'push' hooks. See above for notes on this input. | |
| `repository` | Repository name to push to.<br>Default or empty value represents current github repository. | `${{ github.repository }}` |
| `branch` | Target branch for the push. | `master` |
| `interval` | Time interval (in seconds) between each new check, when waiting for status checks to complete. | `30` |
| `timeout` | Time (in minutes) of how long the action should run before timing out, waiting for status checks to complete. | `15` |
| `sleep` | Time (in seconds) the action should wait until it will start "waiting" and check the list of running actions/checks. This should be an appropriate number to let the checks start up. | `5` |
| `unprotect_reviews` | Momentarily remove pull request review protection from target branch. | `False` |

## License

[MIT License](LICENSE)
