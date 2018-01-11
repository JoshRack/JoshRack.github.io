# Line Endings in Git

## Problem
I've been working on a project where I'm developing locally on my Windows 10 workstation and one of the other developers is on a Linux box.  I noticed some time ago on one of my check-ins that it appeared the whole file had changed. I should've taken this as the warning something was amiss.  Fast-forward several days and my development partner noticed the same thing when he went to commit. When I had committed, the files I had touched had their line endings converted to the Windows CRLF format and he was trying to commit with the Linux LF format.

## Solution
First, here's a command that'll show some history including line endings that had changed:

`for i in {0..5}; do echo; git show HEAD~$i:server/src/index.tsx | file -; git show -s HEAD~$i; done`

And the result:

```/dev/stdin: ASCII text, with CRLF line terminators
commit 7f60690a2dc1e4e1c0387bcab3dbcd8a0adb1026
Author: Josh Rack <JoshuaRack@gmail.com>
Date:   Wed Jan 3 11:10:29 2018 -0600```

```/dev/stdin: Java source, ASCII text
commit 8e0899cc5f5f47dfb21b5d54ded4adff3c6db387
Author: Josh Rack <JoshuaRack@gmail.com>
Date:   Wed Jan 3 09:33:54 2018 -0600```

Notice the `/dev/stdin: ASCII text, with CRLF line terminators`. It was at that check-in that the line endings changed.

We resolved it in two parts. I learned about a `.gitattributes` file here: https://git-scm.com/docs/gitattributes. There's a section called 'End-of-line conversion'. From a clean working directory:

```$ echo "* text=auto" >.gitattributes
$ git read-tree --empty   # Clean index, force re-scan of working directory
$ git add .
$ git status        # Show files that will be normalized
$ git commit -m "Introduce end-of-line normalization"
```

That creates a .gitattributes file with the setting that will enforce line endings to be LF. It then searched and cleaned up all the files to obey that rule.

For certainty, after all this, I made sure to set the following git config: `git config --global core.autocrlf true`
