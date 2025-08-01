---
title: Lint your Bicep and Bicepparam files with GitHub Workflow
date: 2024-01-24 00:00:00 +1000
categories: Azure
tagline: "Linters are a very powerfull tool to validate if your code is correct."
tags:
  - Azure
  - Bicep
  - GitHub
excerpt_separator: <!--more-->
header:
  overlay_image: ./assets/images/Sparrow-2024-Robert-de-Veen.jpg
  caption: "Photo credit: [**Robert de Veen**](https://robertdeveen.com)"
  show_overlay_excerpt: false
---

Linters are a very powerfull tool to validate if your code is correct. With the new `az bicep lint --file $file` command you can validate if your Bicep and Bicepparam files are correct. This command is available in the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/bicep?view=azure-cli-latest#az-bicep-lint) and can be used in a GitHub Action.

<!--more-->

When changing the Bicep files in a pull request, you can use the `az bicep lint --file $file` command to validate the Bicep files. This command will validate the syntax of the Bicep files and will also validate if the resources are valid. This is a great way to validate your Bicep files before deploying them to Azure. 

When adding or removing a parameter from the Bicep template, I often forget to change all the Bicepparam files and when deploying to the Production environment in Azure, the deployment wiil fail because the Bicepparam file isn't correct. To prevent this, I created a GitHub Workflow that will validate the Bicep and the Bicepparam files when a pull request is created or updated. This way, I can't forget to update the Bicepparam files.

## The Bicep Linter GitHub Workflow file

```yaml
name: 🎗️ Bicep Linter

on:
  pull_request:
    types: [opened, reopened, synchronize, edited, ready_for_review]

  workflow_dispatch:

permissions:
  id-token: write
  contents: write
  actions: read

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    name: Lint Bicep and Bicepparam files
    environment: AZURE
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Lint Bicep and Bicepparam files
        run: |
          az bicep install
          success="true"

          CHANGED_FOLDERS=$(git diff HEAD~1 --name-only | xargs dirname)
          echo "📂 Changed folders:"
          echo $CHANGED_FOLDERS

          FILES=$(find $CHANGED_FOLDERS \( -name "*.bicep" -o -name "*.bicepparam" \))
          echo "🔍 Files to lint:"
          echo $FILES

          for file in $FILES; do
          { # 'try' block
            echo "🎗️ Linting $file" && az bicep lint --file $file
          } || { # 'catch' block
            success="false"
          }
          done

          if [ "$success" = "false" ]; then
            echo "::error::❌ Linting of the Bicep files failed."
            exit 1;
          fi;
```

### Checkout code

To only Lint the files that has been changed, and not Lint the entire repository, I use the `actions/checkout@v4` action with the `fetch-depth: 0` parameter to fetch all history for all branches and tags. This way, the `git diff` command can be used to get the changed files.

### Get changed files

With the `git diff HEAD~1 --name-only` command, I get all the changed files in the pull request. The `--name-only` parameter will only return the file names and not the content of the files. The `HEAD~1` parameter will compare the changed files with the main branch. This way, only the changed files in the pull request will be returned.

### Get the changed folder

Now we only have the files that has been changed, with the `xargs dirname` command, we can get the folders of the changed files. Because the Bicep file has change, the Bicepparam files in the same folder should be validated as well.

### Find the Bicep and Bicepparam files

With the `find $CHANGED_FOLDERS \( -name "*.bicep" -o -name "*.bicepparam" \)` command we get all the Bicep and Bicepparam files in the folders that has been changed. The `-o` parameter is a logical OR, so we get all the files that ends with `.bicep` or `.bicepparam`.

### Lint the Bicep and Bicepparam files

Let's Lint the files! With a `for` loop, we can loop through all the files and run the `az bicep lint --file $file` command. The `--file` parameter is the file that needs to be linted.
