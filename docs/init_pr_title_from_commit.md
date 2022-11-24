# 📰 PR Title from first commit

PR titles are initialized generally from the header of the only commit message
of the PR, or it takes the branch name if the log has more than one commit.

Assuming that developers will initialize their branches with a commit (empty or
not) that sums up their changes, it will be good to assign that commit header
to the title of the PR.

This also matches GitHub's convention of single-commit-PRs, behaving similarly.

### How it works

It consists of two basic steps as shown on the image below:

<div align='center'>
 <img src='https://user-images.githubusercontent.com/21087992/177455543-551dcad6-10a7-44ce-bf5c-8bcb5500207f.png' width=500>
</div>

<br />

**Getting the title**

The first one is wrapped into an `echo` command that we'll get to in a minute.
The core of it is on the `curl` statement which gets the list of commits. To
get those commits, we use the GitHub API through the pulls API. Documentation
about the pulls API, particularly the request we are doing here, can be found
[on this link](https://docs.github.com/en/rest/pulls/pulls#list-commits-on-a-pull-request).

After getting pull request's commit data, we access the first commit message
using the `jq` program. Basically, is a way to manipulate JSONs and index them
as you would in any other js application.

```bash
 cat sample.json | jq '.[0] | .commit | .message'
```

The following script will get from the contents of `sample.json` the first
element (`.[0]`), then get the commit attribute (`.commit`) and finally the
message (`.message`). Putting everything together, we get the commit message of
the first commit.

Now that we have the message, we need to pass it to the next step which will
write PR's title. To do so, we wrap everything with the `echo` command:

```yaml
id: get_first_commit
run: |
  echo "commit_message=SOME_SILLY_STRING" >> $GITHUB_OUTPUT
```

From [GitHub's documentation](https://docs.github.com/en/actions/learn-github-actions/contexts#example-usage-of-the-steps-context),
the way to pass the output of a command on GitHub actions to another step, is
by adding the variable to `GITHUB_OUTPUT` as we would in any other env var.
We will go through that in the next step but doing this, we essentially load
this information into the job's runner.

**Writing the title**

This step is much easier because it consists of a simple PATCH request:

```bash
curl \
  -u  ${{ secrets.GH_TOKEN }}:x-oauth-basic \
  -X PATCH \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/${{ github.repository }}/pulls/${{ github.event.pull_request.number }} \
  -d '{"title": ${{ steps.get_first_commit.outputs.commit_message }}}
```

Things to highlight here are first the `github` variables that are available to
use from the context of the workflow that has been triggered. More information
on the documentation that can be found
[here](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context).
However, the TL;DR is that we build the URL to update the PR (as detailed
[here](https://docs.github.com/en/rest/pulls/pulls#update-a-pull-request)) and
the only thing interesting is that we use the output of the previous step.

To do so, we access the environment variable called `steps.get_first_commit.outputs.commit_message`.
In general the syntax will be `steps.<id-of-the-step>.outputs.<name-of-the-variable>`.
Everything was set on the last but one shared snippet, so please check it out to
better understand how they interact.

**Setup**

The last thing to solve the puzzle is how we make it an isolated action to be
triggered on demand.

There are two key settings for that. The first one is the trigger:

```yml
on:
  workflow_call:
```

This event is used to trigger a workflow from another workflow
([documentation](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_call)).
Since we won't be interested in triggering it alone, we only set it to trigger
via this event.

The other configuration happens on the parent, the caller workflow

```yml
edit-pr-title:
    uses: ./.github/workflows/edit-pr-title.yml
    secrets: inherit
```

For the child workflow to access to the repository's secrets, we need to
explicitly tell them to inherit from the parent. This is a nice-and-easy way to
not just inherit the `GH_TOKEN` secret, but any other secret that we might need
so it's a pretty big deal 😅 (for more info,
[this SO post](https://stackoverflow.com/a/72103477) does a good summary of
[GH's docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow)).

|<video alt='Demo featuring a PR whose title is being replaced by the first commit.' src='https://user-images.githubusercontent.com/21087992/177461049-6471189a-a644-49bf-8794-6d61ca18e3a3.mov'> |
| :--: |
| *Demo featuring a PR whose title is being replaced by the first commit.* |
