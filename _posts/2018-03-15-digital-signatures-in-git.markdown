---
layout: post
title:  "Digital signatures in git"
date:   2018-03-15 12:00:00 +0100
categories: git cryptography
description: Why and how to set up digital signatures in git for tags and commits.
---
A digital signature in concept is pretty simple. You want to show two main things: 1) the content that you signed was not changed 2) it was you who signed it.
Those properties can be pretty useful in some cases.

One example for this would be software releases. How can you be sure that the release you're looking at wasn't changed in any way? Without a signature you can't. A Man-in-the-middle could easily change the software while you download it and you wouldn't know unless you could check a signature. (assuming you would check it then)

To mark a release of your software in git you'd use git tags. So logically you should sign all your git tags to provide users the possibility to validate your releases.
To go a step further you can also sign commits in git. In fact you can even start signing every commit. By having that policy that you will always sign every commit of yours, you tell other people that every commit they see from you that is not signed is definitely a forgery. In case you didn't notice until now you can easily create a commit as someone else by changing your git config:

`git config --global user.name 'XXX'`

`git config --global user.email XXX`

So depending on how paranoid you are, you may want to sign all your commits. To do that we need a public/private key pair. If you're totally unfamiliar with GPG or public key cryptography you should take some time to learn a bit about it first, because you will need to take serious measures to protect your private key.

Generate a gpg key if you don't have one yet:

`gpg --gen-key`

when asked for an email make sure it is the same one you're using for your GitHub account.

`gpg --list-secret-keys --keyid-format LONG | grep ^sec`

This will create an output that looks like this:

`sec   rsa4096/<KEY> 2018-03-15 [expires: 2018-03-18]`

Copy the part that says `<KEY>` and use it in the following commands.

`git config --global user.signingkey <KEY>`

This tells git which key to use when signing commits.

`gpg --armor --export <KEY>`

This returns your public key, add this complete block to GitHub: [How to add a public key to GitHub][github-docs]

Finally create your first signed commit that will get a green verified on GitHub:

`git commit -S -m "First signed commit"`

To sign all your future commits in all local repositories (remove `--global` flag to only enable it in the current local repository):

`git config --global commit.gpgsign true`

With this you won't need the `-S` flag anymore.

[github-docs]: https://help.github.com/articles/adding-a-new-gpg-key-to-your-github-account/
