# Documentation

This file consists of the Table of Contents for the Documentation hosted
on this repository. Also, some more general-use documentation is appended here.

## Workflows
- [First Contribution Greeting](./first_contribution_greeting.md)
- [Add Jira Ticket Link To Pr From Branch](./add_jira_ticket_link_to_pr_from_branch.md)

---

## 🏗️ Initial configuration and scaffolding

To make this work, we have a main workflow that's been triggered by the PR and
some global repository configurations to make scripts work. In following
sections, we will go through them.

### 🤐 GitHub Secrets

Some actions require environment variables that is best to keep secret. One of
these variables is the `GITHUB TOKEN`. This token is associated to the personal
account on GitHub, and for these PRs, we need the token of a person with
permission to read and write PRs and commits.

To get your token, go to [this URL](https://github.com/settings/tokens) and create one.

Then, we need to put that token into our repository's secret vault. To do so,
an administrator must go to the repository's setting page and select
`Secrets > Actions`.

In there, you can click the `New repository secret` and create the token called
`GH_TOKEN`. Afterwards you'll see something like this:

<div align='center'>
 <img src='https://user-images.githubusercontent.com/21087992/203157437-60884ac0-2c2a-40eb-9430-6f5bd93137ee.png' width=500>
</div>

With that set, we are good to go to use it on our GitHub Workflows and it won't
show in logs or anything, so it's super secured 💪
