## Description

Add `first_contribution_greeting` workflow to be reused and greet contributors on opening their first PR.

Additionally, they will be reminded by it to follow the contributing guidelines of their repo.

## Usage

### Requirements

Have in your repository a secret created containing your Personal Access Token
for GitHub. If you need guidance on this step, please refer to the main
[README.md](./README.md#GitHub-Secrets) on the docs folder of this repo.

For more info on reusable workflows please check this
[GitHub Doc](https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow).

### Calling the workflow from one of your own

```yaml
...
jobs:
  my-job:
    uses: juanmanso/dx-workflows/.github/workflows/first_contribution_greeting@main
    secrets:
      GH_TOKEN: ${{ secret.NAME_OF_YOUR_LOCAL_SECRET_TOKEN }}
...
```

## Test

| _Test run with new repository_ |
| :---: |
| <video src='https://user-images.githubusercontent.com/21087992/203149044-c12f3166-ac01-413c-8a5a-875f8ef14251.mov'> |
