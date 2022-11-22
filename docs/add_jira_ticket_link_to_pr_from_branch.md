# 🎫 Adding JIRA tickets to PR's descriptions

A part of the completion of the PR is associating the tickets to it, so that
reviewers can have access to the context of the change and some other
information that might be relevant (since JIRA is generally the SSO used by
projects).

Since the work is pretty much a routine, it can be scripted and automated. So
that's what we did! 🥳

The bad thing is that getting the information in an automated and agnostic
manner, might require one or two steps that are a bit confusing 😞

### How it works

This one is gonna get crazy and confusing, but we'll go through it together 💪.
The high-level steps can be determined as:

```
 - Parse branch name to text block
 - Get current PR description
 - Create new description to be assigned
 - Assign new description to PR
```


_Step 1: Parse branch name to text block_

The step is quiet straight forward:

```yml
- name: Parse branch name to text block
  id: parse_branch_name
  run: |
    echo ${{ github.head_ref }} |\
    echo "text_block=$(\
      awk '{n = split($0, t, "/"); for (i = 1; ++i <= n;) printf "- "t[1]" ["t[i]"](${{ inputs.JIRA_BASE_URL }}/browse/"t[i]")\\\\r\\\\n "; print ""}; '
    )" >> $GITHUB_OUTPUT
```

From [GitHub's Docs](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context),
there's a bunch of stuff in that snippet that's only there to pass information
along the next steps.

The core command is:

```bash
echo ${{ github.head_ref }} |\
  awk '{n = split($0, t, "/"); for (i = 1; ++i <= n;) printf "- "t[1]" ["t[i]"](${{ inputs.JIRA_BASE_URL }}/browse/"t[i]")\\\\r\\\\n "; print ""}; '
```

The first part is just to get the source or base branch's name of the pull
request ([documentation](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context)).
Then we process it with the lovely program called `awk` that helps us parse the
branch name, split by `/` and iterate through the results outputting the bullets
that will be shown on the PR itself.

> Little comment: the `\\\\r\\\\n` should be `\\r\\n` instead. However, since
> we are going to insert this into another command, we need to scape every `\`
> to avoid misunderstanding them.
>
> Hope this comment makes things clearer 😅

As a side-note, we are using `inputs` on GitHub Workflows to parametrize JIRA's
base URL. When this workflow is called, it should be provided to make it work.

<div align='center'>
 <img src='https://user-images.githubusercontent.com/21087992/177463461-a927fe8d-7051-4088-b77e-fac953d0956f.png'>
</div>

<br />

_Step 2: Get current PR description_

Since the developer might have edited the template before submitting the new PR,
we need to preserve its contents and just add the new bullets. Thus, we must
get the current state of the description.

We get PR's info from GitHub's API and then index the resulting JSON with `jq`:

```yml
- name: Get current PR description
  id: get_current_pr_description
  run: |
    echo "current_body=$(\
      curl \
        -u  ${{ secrets.GH_TOKEN }}:x-oauth-basic \
        -H "Accept: application/vnd.github.v3+json" \
        https://api.github.com/repos/${{ github.repository }}/pulls/${{ github.event.pull_request.number }} \
        | jq '.body'\
    )" >> $GITHUB_OUTPUT
```

_Step 3: Create new description to be assigned_

On this step, we merge the result of the first and second step. It might look
complicated

```yml
- name: Create new description to be assigned
  id: create_new_description
  run: |
    echo ${{ steps.get_current_pr_description.outputs.current_body }} |\
    echo "new_body=$(\
     awk '{ \
        n = split($0, t, "${{ inputs.DELIMITER }}"); \
        printf "%s ${{ steps.parse_branch_name.outputs.text_block }}\\r\\n${{ inputs.SCAPED_DELIMITER }} %s", t[1], t[2];\
      }'\
    )" | sed s/\'/\'\\\\\'\'/g >> $GITHUB_OUTPUT
```

but it's just:

```bash
echo ${{ steps.get_current_pr_description.outputs.current_body }} |\
 awk '{ \
    n = split($0, t, "${{ inputs.DELIMITER }}"); \
    printf "%s ${{ steps.parse_branch_name.outputs.text_block }}\\r\\n${{ inputs.SCAPED_DELIMITER }} %s", t[1], t[2];\
  }'
```

First we get the current state of the body (acquired on the step number 2).
Then we split the giant string (it has their breaklines marked with `\n`'s) by
the delimiter. Secondly, we concat both strings and add the new bulleted list
in the middle.

> Reminder of `printf`: by using `%s` we tell `printf` that in there it will be
a string and to fill it with the first variable given (`t[1]`). The second time
the `%s` occurs, the same happens but with the second variable (`t[2]`).

Lastly, with `sed` we cast and wrap `'` in a pair of single-quotes resulting in
 `'\''` that makes `curl` command work.

_Step 4: Assign new description to PR_

Now that we have our PR body ready to be used, we just do the PATCH request
with the new body!

```yml
run: |
  echo ${{ steps.create_new_description.outputs.new_body }} \
  curl \
    -u  ${{ secrets.GH_TOKEN }}:x-oauth-basic \
    -X PATCH \
    -H "Accept: application/vnd.github.v3+json" \
    https://api.github.com/repos/${{ github.repository }}/pulls/${{ github.event.pull_request.number }} \
    -d '{"body": "${{ steps.create_new_description.outputs.new_body }}"}'
```

Now that we get how it works, let's see it in action! 🚀

|<video alt="Demo featuring a PR that gets their liked JIRA's from branch's name." src='https://user-images.githubusercontent.com/21087992/203195701-9a33b207-4b07-482f-99f1-08a62d36cce5.mov'> |
| :--: |
| *Demo featuring a PR that gets their linked JIRA's from branch's name.* |
