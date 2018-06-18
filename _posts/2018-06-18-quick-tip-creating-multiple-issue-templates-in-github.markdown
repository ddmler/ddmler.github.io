---
layout: post
title:  "Quick Tip: Creating multiple issue templates in GitHub"
date:   2018-06-18 18:00:00 +0100
categories: github
description: How to offer different issue templates before a user creates a new issue in your GitHub project.
---

The possibility to create issue or pull request templates in GitHub is nothing new. However it is also possible to create multiple templates and offer a nice view to the user that lets him choose a suitable template before creating a new issue.

To do this, create a new file `.github/ISSUE_TEMPLATE/bug.md`:

```markdown
---
name: Bug report
about: Create a new bug report
---

Template goes here.
```

The name and about attributes will be shown to the user before he creates a new issue. If you want some more issue templates just create new files in this directory. At the end it could look like the issue template chooser of facebook/jest:

![Example of Facebooks Jest issue templates](assets/github_issues.PNG)

They have 4 different templates and if nothing seems to fit for your issue you can still choose to have no template. You can try it out on [Jests GitHub page][jest]. Create a new issue (but don't submit it) to see how the template choosing works.

The same is possible for pull requests, however there isn't a nice page offering the templates and instead it seems you have to use it as a query string in the url by adding:

`?quick_pull=1&template=bug.md` to use the template at `.github/PULL_REQUEST_TEMPLATE/bug.md`

[jest]: https://github.com/facebook/jest
