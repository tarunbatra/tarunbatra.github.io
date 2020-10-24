---
title: Git before Github
date: 2020-10-24 22:17:54
tags:
  - git
  - github
  - opensource
category:
photos: /data/images/git-before-github/cover.png
description: Have you collaborated over git without using platforms like Github? I haven't either. But git is distributed in itself so it is possible and here is how.
---

Back when I was studying CS at university, we had assignments to code various algorithms. When the assignments were due we were supposed to execute the code in laboratory systems and get the output checked by the professors. I liked coding and tinkering with the algorithms; some of my classmates didn't. They used to copy the code from my system on a USB stick and execute them on their system and get done with it. I didn't judge them but I was tired of giving each one their copy and sometimes also instructing them how to execute that piece of code. In one such assignment, I uploaded the code on GitHub and shared the [link][re2nfa-repo] with anybody who asked. __Unknowingly, I had made my first open source contribution.__

A lot of people have similar stories and for most, Github is synonymous with Open Source. One thing that is also used interchangeably with Github is git. Even though these are three different entities, a lot of developers including me have never contributed to open source or used git without Github. Thus I was surprised when I learned that famous open source projects like Linux were not on Github even though they used git. I wondered how did it work but didn't do anything about it until recently when I decided to experience a git development workflow without Github.

## Git server access
We use version control systems broadly for two use cases -- to contribute to a repository where we have write access; to contribute to a public repository where we don't have write access. To collaborate with developers to whom access can be given, Github or other platforms are not necessary. Anybody can host a [Git server][git-server-guids] that can be used to host repositories. Developers can clone, push to, and pull from these remote repositories once access is granted to their ssh identity.

```
git clone origin git@<host>/<repo>.git
```

This is similar to how we upload our SSH keys to Github or other platforms for being able to clone the repositories using SSH.

## Pull requests
One thing which is missing in the above-mentioned method is the ability to create Pull Requests, get them reviewed and merged into the code. It becomes a big issue when the developers don't have write access to the repositories as is the case in open source development. General open source flow is to fork a repository, push the changes in your forked repositories, and create a __pull request__ to the original repository from where the maintainers can review and merge. Pull request is not a feature of git, but of the platforms like Github. A similar workflow in git can be done by:
1. Clone the original repository
2. Create a patch
3. Send the patch to the maintainer
4. The maintainer applies the patch and pushes the code

We introduced two new processes in this workflow -- creating a patch and applying a patch. Let's discuss them further.

---
### Patch
A patch is a diff of code and metadata around it. A diff is the actual code change which can be viewed using the command `git diff`. It is a Unix concept that is way older than git and Unix systems even have a builtin tool, [`diff`][diff-man-page] which compares changes between two files. Its output can then be saved to a file that can be processed by another utility tool called `patch` which updates one of the files to make its content identical to another file.
```sh
touch original.txt new.txt
echo "New addition" > new.txt
```
```sh
diff original.txt new.txt -u > changes.patch
```
```patch changes.patch
--- original.txt	2020-10-18 23:43:32.000000000 +0200
+++ new.txt	2020-10-18 23:43:32.000000000 +0200
@@ -0,0 +1 @@
+New addition
```
If we want to update the original file with the contents of the new one, we need to "patch" it.
```sh
patch original.txt changes.patch
```
__NOTE:__ _Use [git-before-github repository][git-before-github-repo] to practice this example and all the ones which are to follow._

Now that we understand the concepts of diff and patch, we can proceed with using them in git. To generate a patch file in git, we need to use `git format-patch` command.
```
git format-patch HEAD~1..HEAD
```

This will create a patch file for the latest commit. Tinker with the argument to produce patch files for other commits too. By default, each file represents a single commit and the file name is of the format `0001-<commit message>.patch`. Interestingly, it is in [Unix mbox][unix-mbox-wiki] format which means it has some email-like metadata (from, subject, etc.) followed by the patch data.
```patch 0001-Added-Tarun-Batra-to-contributors.patch
From 619814a3ac21012580e725136864e8397c14e20b Mon Sep 17 00:00:00 2001
From: Tarun Batra <tarun.batra00@gmail.com>
Date: Tue, 20 Oct 2020 23:15:32 +0200
Subject: [PATCH] Added Tarun Batra to contributors

---
 CONTRIBUTORS.txt | 1 +
 1 file changed, 1 insertion(+)

diff --git a/CONTRIBUTORS.txt b/CONTRIBUTORS.txt
index e69de29..1d12219 100644
--- a/CONTRIBUTORS.txt
+++ b/CONTRIBUTORS.txt
@@ -0,0 +1 @@
+Tarun Batra <tarun.batra00@gmail.com>
--
2.28.0
```
This is the patch file I created to add my name to the contributors' list of the [git-before-github][git-before-github-repo] repository as an exercise for myself.

---
### Apply
We have a patch file which we can send to the maintainers so that they can apply the patch. We can either use traditional ways to send the patch or use git itself, but more on it later. In this section, we will explore the process to apply a patch.

`git apply <patch file>` is a command which applies the changes described in the patch file locally but does not commit them. They will appear in our workspace and we can stage them and commit them. This is not ideal as even though we get the changes, the commit metadata like author name and message is lost. A better way to apply patches is `git am` (abbrev. for "apply mailbox").
```
git am --sign-off <patch file>
```
The flag `--sign-off` is used if the commit message needs to be appended with a "Signed-off-by" line to indicate who applied the commit. Below is the signed-off and applied commit for the patch we generated as an exercise in the last section.
```sh git log
Author: Tarun Batra <tarun.batra00@gmail.com>
Date:   Tue Oct 20 23:15:32 2020 +0200

    Added Tarun Batra to contributors

    Signed-off-by: Tarun Batra <tarun.batra00@gmail.com>
```
---
### Sending the patch
Now we can explore how to send a patch file to a maintainer in the "git" way. There are two ways that I know of --

1. imap-send
```sh
git imap-send <patch file>
```
It will upload the email to an IMAP folder from where it can be sent. To use it we need to add IMAP server details in our `~/.gitconfig` file. I use Gmail so my IMAP details look like:
```ini
[imap]
	folder = "[Gmail]/Drafts"
	host = imaps://imap.gmail.com
	user = email@gmail.com
	pass = yourpassword
	port = 993
```
Gmail allows its IMAP server to be used if the [Less secure app access][google-less-secure-setting] setting is turned on.

1. send-mail
```sh
git send-mail --to=<email> <patch file>
```
It will use an SMTP server to send email. The corresponding config to use this command is:
```ini
[sendemail]
	smtpEncryption = tls
	smtpServer = <host>
	smtpUser = <user>
	smtpServerPort = 587
```
Both of these methods can be used to send an email per patch file. The subject of the commit by default is of the format `[PATCH] <commit message>`.

__NOTE:__ _Sender email addresses can be spoofed quite easily and this raises the question of authentication and authorization when submitting patches. I tried creating patches of PGP signed git commits but the patch doesn't retain the PGP signature. We could encrypt the email itself using PGP but that's a lot of extra work for both the contributor and the maintainer._

So what can we do with this newly acquired knowledge?
1. Git patches often [float][lkml-1] [around][lkml-2] in the Linux development mailing list and it gives me satisfaction that I understand how they work a little better now than before.
2. When platforms like Github and Gitlab have an outage, work in most of the development teams stall. Now I know a way to get my code reviewed even in these situations. (Okay, that might be a stretch)

You can practice for yourself by adding your name to the `CONTRIBUTORS.txt` file of the [git-before-github][git-before-github-repo] repository, committing it, and then sending me the patch. I would readily ~merge~ apply it. 😀

[re2nfa-repo]: https://github.com/tarunbatra/re2nfa
[git-server-guide]: https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server
[diff-man-page]: https://man7.org/linux/man-pages/man1/diff.1.html
[git-before-github-repo]: https://github.com/tarunbatra/git-before-github
[unix-mbox-wiki]: https://en.wikipedia.org/wiki/Mbox
[google-less-secure-setting]: https://myaccount.google.com/lesssecureapps
[lkml-1]: https://lkml.org/lkml/2020/10/16/629
[lkml-2]: https://lkml.org/lkml/2020/10/19/82

