# Readme Download Button Action

GitHub Action workflow configuration for keeping a direct download link of the latest version in your README.md file.

## Example

With a single click of the button below, a zip of this repository start downloading.

<!-- BEGIN LATEST DOWNLOAD BUTTON -->
[![Download zip](https://custom-icon-badges.herokuapp.com/badge/-Download-blue?style=for-the-badge&logo=download&logoColor=white "Download zip")](https://github.com/DenverCoder1/readme-download-button-action/archive/1.0.0.zip)
<!-- END LATEST DOWNLOAD BUTTON -->"
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
