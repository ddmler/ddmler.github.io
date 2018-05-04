---
layout: post
title:  "Finding credential leaks to version control with gitleaks"
date:   2018-05-04 18:00:00 +0100
categories: git
description: Gitleaks is a static analysis tool that can scan your git repos for leaked credentials like AWS access keys.
---

[Gitleaks][gitleaks] is a static analysis tool that can scan your git repos for leaked credentials like AWS access keys. These kinds of leaks are a fast growing attack vector due to the increasing usage of such keys so it is a good idea to add a tool like this to your CI process to catch these mistakes as fast as possible.

You can get it via the go package manager:
`go get -u github.com/zricethezav/gitleaks`

Or download the binary from here: [Gitleaks Releases][releases]. After that just run:

`gitleaks`

This will run the audit on the current working directory. Or

`gitleaks --temp https://github.com/some/repo`

will clone the repo from GitHub to a temporary folder, run the audit and delete the folder after that. You can also use it with docker:

`docker run --rm --name=gitleaks zricethezav/gitleaks https://github.com/some/repo`

In case the tool found a valid leak you should follow [GitHubs Guide to remove sensitive data][github].


[gitleaks]: https://github.com/zricethezav/gitleaks
[releases]: https://github.com/zricethezav/gitleaks/releases
[github]: https://help.github.com/articles/removing-sensitive-data-from-a-repository/
