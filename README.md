# github-action-module-release

モジュールリポジトリの リリースタグと Github Release の作成を自動化するための Action 郡

コミットメッセージを [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) に沿ったものにすることで､ git tag と Github Release (リリースノート付き) を作成する｡ 各開発者にコミットメッセージを強要するのは難しいので PR タイトルを Conventional Commits specification に沿ったものにし､ Squash merge することで実現する

# Workflows

## .github/actions/check-pr-title

[Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/)に沿ったPRタイトルの名付けになっているかを確認する Action

### Env

| name         | description     | ex                          |
|--------------|-----------------|-----------------------------|
| GITHUB_TOKEN | Githubのアクセストークン | ${{ secrets.GITHUB_TOKEN }} |

## .github/actions/create-release-pr-for-gem

gem リポジトリのリリース PR を作成する Action｡ gem は version 値を `{モジュール定数}::VERSION` に書かなければならないので PR 経由が必要｡

### Inputs

| name                | description                   | default                                                 | ex                        |
|---------------------|-------------------------------|---------------------------------------------------------|---------------------------|
| base-branch         | リリース対象のブランチ名                  |                                                         | `main`                    |
| version-file-path   | gem の version.rb のパス          |                                                         | `./lib/my_gem/version.rb` |
| commit-user-email   | version.rb の変更コミットのユーザー Email | `41898282+github-actions[bot]@users.noreply.github.com` |                           |
| commit-user-name    | version.rb の変更コミットのユーザー名      | `github-actions[bot]`                                   |                           |
| tag-prefix | git tag のプレフィックス                | `v`                                                     | `ruby-gems`               |

### Envs 

| name         | description     | ex                          |
|--------------|-----------------|-----------------------------|
| GITHUB_TOKEN | Githubのアクセストークン | ${{ secrets.GITHUB_TOKEN }} |


## .github/actions/release-by-merge

PR マージで git tag と Github Release を作成する Action｡ ファイルの変更が不要なモジュールで利用する (ex. Go module)｡

### Envs

| name         | description     | ex                          |
|--------------|-----------------|-----------------------------|
| GITHUB_TOKEN | Githubのアクセストークン | ${{ secrets.GITHUB_TOKEN }} |


## .github/actions/release-by-release-pr

Release PR のマージで git tag と Github Release を作成する Action｡ ファイルの変更が必要なモジュールは PR を経由する必要があるのでこちらを利用する (ex. Ruby gem)｡

### Inputs

| name                | description                   | default                                                 | ex                       |
|---------------------|-------------------------------|---------------------------------------------------------|--------------------------|
| base-branch         | リリース対象のブランチ名                  |                                                         | `main`                   |

### Envs

| name         | description     | ex                          |
|--------------|-----------------|-----------------------------|
| GITHUB_TOKEN | Githubのアクセストークン | ${{ secrets.GITHUB_TOKEN }} |



# Usage

## Repository Settings

1. Settings > General > Pull Requests で Allow merge commits のチェックを外し､ Allow squash merging のチェックを入れつつ Default commit message を Pull Request title に選択する
2. dependabot の設定がある場合は `commit-message.prefix` に `ci(bundler):` などの Conventional Commits specification にそった prefix を設定する

## Action Settings

```yml
# example
name: Create release PR

on:
  workflow_dispatch:

jobs:
  create_pr:
    runs-on: ubuntu-latest
    steps:
      - uses: pinnacles/github-action-module-release/.github/actions/create-release-pr-for-gem@v1.0.0
        with:
          base-branch: master
          version-file-path: ./lib/my_gem/version.rb
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```



