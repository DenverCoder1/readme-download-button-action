# Readme Download Button Action

GitHub Action workflow configuration for keeping a direct download link of the latest version in your README.md file.

## Example

With a single click of the button below, a zip of this repository will start downloading.

<!-- BEGIN LATEST DOWNLOAD BUTTON -->
[![Download zip](https://custom-icon-badges.herokuapp.com/badge/-Download-blue?style=for-the-badge&logo=download&logoColor=white "Download zip")](https://github.com/DenverCoder1/readme-download-button-action/archive/1.0.1.zip)
<!-- END LATEST DOWNLOAD BUTTON -->

The URL this button leads to is automatically updated on each release by this [workflow](.github/workflows/download-button.yml).

## Basic Usage

This workflow consists mainly of four parts:

- Checkout the latest version of the repo with [actions/checkout](https://github.com/actions/checkout)
- Get the latest release with [InsonusK/get-latest-release](https://github.com/InsonusK/get-latest-release)
- Insert a download button for the latest version in the readme using a custom shell script
- Update the readme with [EndBug/add-and-commit](https://github.com/EndBug/add-and-commit)

### Step 1

Add the following snippet within your README.md file anywhere you want the button to appear:

```md
<!-- BEGIN LATEST DOWNLOAD BUTTON -->
<!-- END LATEST DOWNLOAD BUTTON -->
```

### Step 2

Create a workflow by placing the following in a `.yml` file in your `.github/workflows/` directory:

```yml
name: "Download Button Action"

on:
  release:
    types:
      - published
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Get latest release
        id: get-latest-release
        uses: InsonusK/get-latest-release@v1.0.1
        with:
          myToken: ${{ github.token }}
          view_top: 1

      - name: Readme Download Button Action
        env:
          GITHUB_USER: "DenverCoder1"
          REPO: "readme-download-button-action"
          FORMAT: "zip"
          VERSION: "${{ steps.get-latest-release.outputs.tag_name }}"
          COLOR: "blue"
          BEGIN_TAG: "<!-- BEGIN LATEST DOWNLOAD BUTTON -->"
          END_TAG: "<!-- END LATEST DOWNLOAD BUTTON -->"
        run: |
          UPDATE=$(cat README.md | perl -0777 -pe 's#(${{ env.BEGIN_TAG }})(?:.|\n)*?(${{ env.END_TAG }})#${1}\n[![Download ${{ env.FORMAT }}](https://custom-icon-badges.herokuapp.com/badge/-Download-${{ env.COLOR }}?style=for-the-badge&logo=download&logoColor=white "Download ${{ env.FORMAT }}")](https://github.com/${{ env.GITHUB_USER }}/${{ env.REPO }}/archive/${{ env.VERSION }}.${{ env.FORMAT }})\n${2}#g')
          echo "${UPDATE}" > README.md

      - uses: EndBug/add-and-commit@v7
        with:
          message: "docs(readme): Bump download button version to ${{ steps.get-latest-release.outputs.tag_name }}"
          default_author: github_actions
          branch: main
```

### Step 3

Update the variables in the `env` section of the `Readme Download Button Action` step to include your username and repo along with any configuration you want as described below in the [Options](#options) section.

## Options

In the `Readme Download Button Action` step of the workflow, the following environment variables must be set:

| Variable      | Example                                 | Description                                             |
| ------------- | --------------------------------------- | ------------------------------------------------------- |
| `GITHUB_USER` | "DenverCoder1"                          | The GitHub username of the user who owns the repository |
| `REPO`        | "readme-download-button-action"         | The name of the repository                              |
| `FORMAT`      | "zip", "tar.gz"                         | The file format to download (`zip` or `tar.gz`)         |
| `VERSION`     | "0.0.1", `<tag of latest release>`*     | The tag name of the release                             |
| `COLOR`       | "blue", "0088AA"                        | The color of the download button (name or hex code)     |
| `BEGIN_TAG`   | `<!-- BEGIN LATEST DOWNLOAD BUTTON -->` | The beginning tag of the download button                |
| `END_TAG`     | `<!-- END LATEST DOWNLOAD BUTTON -->`   | The ending tag of the download button                   |

*For using the latest release, include the `get-latest-release` step and set `VERSION` to `${{ steps.get-latest-release.outputs.tag_name }}`

## Advanced Usage

This workflow can also be used in combination with other actions for automatically creating tags and releases in addition to updating the readme.

For example:

- Calculate the next version number based on the latest release with a shell script or an action for [Semantic Versioning](https://semver.org/)
- Create a tag with [tvdias/github-tagger](https://github.com/tvdias/github-tagger)
- Create a release with a changelog using [fregante/release-with-changelog](https://github.com/fregante/release-with-changelog)


```yml
name: "Automatic Release"

on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Get latest release
        id: get-latest-release
        uses: InsonusK/get-latest-release@v1.0.1
        with:
          myToken: ${{ github.token }}
          view_top: 1
      
      - name: Get next tag
        id: next-tag
        run: |
          prev_version=$(echo "${{ steps.get-latest-release.outputs.tag_name }}" | sed -E 's/\..*//g')
          echo "Previous version: $((prev_version))"
          echo "Next version: $((prev_version+1))"
          echo "::set-output name=tag::$((prev_version+1)).0"
          
      - name: Create tag
        uses: tvdias/github-tagger@v0.0.1
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          tag: ${{ steps.next-tag.outputs.tag }}

      - uses: fregante/release-with-changelog@v3
        id: release-with-changelog
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          exclude: '^meta|^docs|^document|^lint|^ci|^refactor|readme|workflow|bump|dependencies|yml|^v?\d+\.\d+\.\d+'
          tag: ${{ steps.next-tag.outputs.tag }}
          title: 'Version ${{ steps.next-tag.outputs.tag }}'
          commit-template: '- {hash} {title}'
          skip-on-empty: true
          template: |
            ### Changelog

            {commits}

            {range}
            
      - name: Delete tag if release skipped
        if: ${{ steps.release-with-changelog.outputs.skipped == 'true' }}
        run: |
          git tag -d ${{ steps.next-tag.outputs.tag }}
          git push origin :refs/tags/${{ steps.next-tag.outputs.tag }}

      - name: Readme Download Button Action
        if: ${{ steps.release-with-changelog.outputs.skipped == 'false' }}
        env:
          GITHUB_USER: "DenverCoder1"
          REPO: "readme-download-button-action"
          FORMAT: "zip"
          VERSION: "${{ steps.next-tag.outputs.tag }}"
          COLOR: "blue"
          BEGIN_TAG: "<!-- BEGIN LATEST DOWNLOAD BUTTON -->"
          END_TAG: "<!-- END LATEST DOWNLOAD BUTTON -->"
        run: |
          UPDATE=$(cat README.md | perl -0777 -pe 's#(${{ env.BEGIN_TAG }})(?:.|\n)*?(${{ env.END_TAG }})#${1}\n[![Download ${{ env.FORMAT }}](https://custom-icon-badges.herokuapp.com/badge/-Download-${{ env.COLOR }}?style=for-the-badge&logo=download&logoColor=white "Download ${{ env.FORMAT }}")](https://github.com/${{ env.GITHUB_USER }}/${{ env.REPO }}/archive/${{ env.VERSION }}.${{ env.FORMAT }})\n${2}#g')
          echo "${UPDATE}" > README.md

      - uses: EndBug/add-and-commit@v7
        if: ${{ steps.release-with-changelog.outputs.skipped == 'false' }}
        with:
          message: 'docs: Bump version to ${{ steps.next-tag.outputs.tag }}'
          default_author: github_actions
```

## License

This work is under an [MIT license](LICENSE)

## Support

If you like this project, give it a ⭐ and share it with friends!
