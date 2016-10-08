---
layout:     post
title:      git-journal 📖
date:       2016-10-08
summary:    A git workflow extension written in Rust. Or why Rust is your language of choice for solving common problems.
categories: git
---

# Your documentation is crap!
Documenting things is one of the hardest things during development. Finding variable names, describing how to use
something can sometimes feels like a waste of time. But every good developer will agree that the first rule should be to
write code that your successor will love it. The same rule applies to the documentation: It is just satisfying if the
new team member reads and understands your code instead of annoying you and others. This also has an economical
impact, for sure.

One very important part of the documentation is the (git) commit message history since it refers to a usual workflows
like `git blame` or `git bisect`. Viewing the `git log` for getting up to date if you were on vacation is just another
scenario. If your team does not follow a commit message rule set then it will be hard to get necessary information from
the commit message. I already had my "what the fuck" moments if I just looked at commit messages like `Fixed build`,
`Fixed build again` and `Finally fixed it`.

Usually, maintaining a separate changelog file should not be needed at all. The necessary information are already
available within the git log. But as already mentioned, they are often just crap. This means we maintain an extra file
for the history of the product, which will be messed up by merge conflicts, wrong or completely missing entries.

# Trash your changelog file!
Alright, [git-journal](https://github.com/saschagrunert/git-journal) tries to solve all these problems mentioned above.
It will prepare your commit and verify it after your edit. If your commits follow the defined rules it will be possible
to generate a changelog from your commits. It is also possible to define a toml based template for your changelog, which
will enable you to categorize even single entries within the commit message.



