# Host NuGet on GitHub

[![Test](https://github.com/moomiji/host-nuget-on-github/actions/workflows/test.yml/badge.svg)](https://github.com/moomiji/host-nuget-on-github/actions/workflows/test.yml)

This action generates a static NuGet v3 feed and pushes it as an orphan branch to a Github repository.

Open the feed via GitHub-Pages or https://github.com/{repository}/raw/{branch}/{feed-path}/index.json.

> Thanks to [Sleet](https://github.com/emgarten/Sleet), the static feed generator, for being such a cool tool!

## Usage

```yml
- uses: moomiji/host-nuget-on-github@v1 # Do not use @main
  with:
    # Specify a URI to write to the feed json files instead of the container's URI.
    # Useful if serving up the content from a different endpoint.
    base-uri: # required

    # Relative path to individual packages or directories containing packages.
    package-paths: # required

    # Bash commands executed on sleet pushed.
    on-sleet-pushed: ''

    # Relative path under $GITHUB_WORKSPACE/.. to place the nuget feed repository.
    # The environment variable $FEED_WORKSPACE is set to its absolute path.
    # Useful if specifying a config.
    repository-path: 'GitHub-Hosted'

    # [Relative path under $FEED_WORKSPACE/ to the output directory of the feed.](https://github.com/emgarten/Sleet/blob/main/doc/client-settings.md#folder-feed-specific-properties)
    feed-path: ''

    # [Relative path to sleet.json where the source information is contained.](https://github.com/emgarten/Sleet/blob/main/doc/commands.md#push)
    config: 'None'

    # [Overwrite existing packages.](https://github.com/emgarten/Sleet/blob/main/doc/commands.md#push)
    force: false

    # [Skip packages that already exist on the feed.](https://github.com/emgarten/Sleet/blob/main/doc/commands.md#push)
    skip-existing: false

    # Message to use when committing changes.
    commit-message: '${{ github.repository }}(${{ github.ref_name }})'

    # Author name and email address in the format `Display Name <email@address.com>`.
    commit-author: '${{ github.actor }} <${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com>'

    # User name of the commiter.
    commit-user-name: 'github-actions[bot]'

    # User email address of the commiter ( {user.id}+{user.login}@users.noreply.github.com ).
    # See [users API](https://api.github.com/users/github-actions%5Bbot%5D).
    commit-user-email: '41898282+github-actions[bot]@users.noreply.github.com'

    # Sign commits use `web-flow.gpg` or `commit-user-gpg-key` of your own bot.
    commit-sign: false

    # GPG key of your own bot. Defaults to `web-flow.gpg`.
    commit-user-gpg-key: ''

    # Passphrase of the GPG key.
    commit-user-gpg-passphrase: ''

    # Indicate whether to squash all commits of the branch.
    squash: false

    # Repository name with owner. For example, actions/checkout
    repository: ''

    # The branch name. Otherwise, uses the default branch.
    branch: ''

    # Personal access token (PAT) used to fetch and push the repository.
    token: ${{ github.token }}

    # Whether to download Git-LFS files.
    lfs: false
```

### Token permissions

To ensure your GitHub Actions workflows function correctly, it's important to configure the `token` with necessary permissions `contents: write`.

- If the nuget feed repository ___is___ the repository using this Action

Please read [ this article](https://github.com/ad-m/github-push-action#requirements-and-prerequisites), or add the following code directly to the job:

```yml
jobs:
  job:
    permissions:
      contents: write
```

- If the nuget feed repository ___is not___ the repository using this Action

Please visit [the link](https://github.com/settings/personal-access-tokens/new), create the PAT token with `Read and write` access to the nuget feed repository, add the token to the repository's [secrets](https://github.com/owner/repo/settings/secrets/actions), and use secret like the following code:

```yml
- uses: moomiji/host-nuget-on-github@v1 # Do not use @main
  with:
    # ...
    token: ${{ secrets.<secret_name> }}
```

### Action outputs

The following outputs can be used by subsequent workflow steps.

- `feed-uri` - Full uri of the feed's root.

Step outputs can be accessed as in the following example.
Note that in order to read the step outputs the action step must have an id.

```yml
      - id: nuget
        uses: moomiji/host-nuget-on-github@v1
      - if: ${{ steps.nuget.outputs.feed-uri }}
        run: |
          echo "feed-uri - ${{ steps.nuget.outputs.feed-uri }}"
          curl -fSL "${{ steps.nuget.outputs.feed-uri }}"
```

## FAQ

- About contributing

If you find a bug or have a feature request, please open an issue on the GitHub repository.
If you want to contribute code, feel free to fork the repository and submit a pull request from `v1` to `main`.

- About signing commits as "github-actions[bot]"

Unavailable. There is no suitable signing action for a "composite" action, and I also don't want to waste too much effort on this feature.

- How to enable the symbol store

The action just uses `sleet push` command, it will create a new feed with default setting when there is no feed.
So if the symbol store is needed, please read [the article](https://github.com/emgarten/Sleet/blob/main/doc/symbol-server.md) and set it up on the branch.
