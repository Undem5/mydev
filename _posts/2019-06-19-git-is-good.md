---
category: CTFLearn
path: '/ctf'
title: 'Git is good'

layout: nil
---

### Git

I've solve an interesting CTF: recover a git repository. This is a good challenge if you want to understand a little more how Git works.
So how does Git work? (The bible is here [https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain]
* Firstly we have a ```.git ``` directory after we init the repository on your local system
* The ```objects``` directory is where all the contents of the database can be found
** The objects appeared to be hashed (SHA-1) ``` .git/objets/[first 2 bytes]/[last 38 bytes]
* ```refs``` is a SHA1 identifier that refers to a commit object.
* ```HEAD``` show the current branch and refers to the most recent commit on the current branch.

We have 3 kinds of objets: 
* commit object
* tree objects
* blob objects

Every commit holds a tree, linked to one or more parents.
Every tree contains other trees and blobs in its leaves.
Blob objects which are the data/source code 

It may be a good entry point to see the history of the repository.

### Walkthrough

So If I do :
```
root@Host:~/Téléchargements/gitIsGood/.git# cat refs/heads/master 
d10f77c4e766705ab36c7f31dc47b0c5056666bb

root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p d10f77c4e766705ab36c7f31dc47b0c5056666bb
tree 05eb4df96314881123c371fccb7cc11fa9a9dba9
parent 195dd65b9f5130d5f8a435c5995159d4d760741b
author LaScalaLuke <lascala.luke@gmail.com> 1477852398 -0400
committer LaScalaLuke <lascala.luke@gmail.com> 1477852398 -0400

Edited files

root@Host:~/Téléchargements/gitIsGood/.git# git log --pretty=oneline
d10f77c4e766705ab36c7f31dc47b0c5056666bb (HEAD -> master) Edited files
195dd65b9f5130d5f8a435c5995159d4d760741b Edited files
6e824db5ef3b0fa2eb2350f63a9f0fdd9cc7b0bf edited files

root@Host:~/Téléchargements/gitIsGood/.git# 
```

We can see the HEAD is pointing to the last commit which is indicated in the logs.
The command ``` git cat-file -p HASH``` allows us to see the content of an object

So we can see from the logs the owner edtied three times the files. Let's check the different versions
We need to see the content of the tree object and then the blob object If we want to find the flag.

```
root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p d10f77c4e766705ab36c7f31dc47b0c5056666bb
tree 05eb4df96314881123c371fccb7cc11fa9a9dba9
parent 195dd65b9f5130d5f8a435c5995159d4d760741b
author LaScalaLuke <lascala.luke@gmail.com> 1477852398 -0400
committer LaScalaLuke <lascala.luke@gmail.com> 1477852398 -0400

Edited files
root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p 05eb4df96314881123c371fccb7cc11fa9a9dba9
100644 blob c5250d0c96b529c78191ac000f85ca165585e73c	flag.txt

root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p c5250d0c96b529c78191ac000f85ca165585e73c
flag{REDACTED}

root@Host:~/Téléchargements/gitIsGood/.git# cat ../flag.txt 
flag{REDACTED}
```
So we looked into the tree of the last commit. We got a blob object corresponding to our file flag.txt. Looking into this file shows us that the owner modify the content of the flag to flag{REDACTED} like we see in the file we have.
So we need to look before:

```
root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p 195dd65b9f5130d5f8a435c5995159d4d760741b
tree 7418780824c88883006d6b6084fc3550c5b9bc3f
parent 6e824db5ef3b0fa2eb2350f63a9f0fdd9cc7b0bf
author LaScalaLuke <lascala.luke@gmail.com> 1477852364 -0400
committer LaScalaLuke <lascala.luke@gmail.com> 1477852364 -0400

Edited files
root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p 7418780824c88883006d6b6084fc3550c5b9bc3f
100644 blob 8684e6896a395765ebadbdf5959657f5541e4b4b	flag.txt

root@Host:~/Téléchargements/gitIsGood/.git# git cat-file -p 8684e6896a395765ebadbdf5959657f5541e4b4b
flag{protect_your_git} 
```

So we look into the second commit and repeat the same actions and found the flag.

Another way if you're lazy:

```

root@Host:~/Téléchargements/gitIsGood/.git# git show 195dd65b9f5130d5f8a435c5995159d4d760741b
commit 195dd65b9f5130d5f8a435c5995159d4d760741b
Author: LaScalaLuke <lascala.luke@gmail.com>
Date:   Sun Oct 30 14:32:44 2016 -0400

    Edited files

diff --git a/flag.txt b/flag.txt
index c5250d0..8684e68 100644
--- a/flag.txt
+++ b/flag.txt
@@ -1 +1 @@
-flag{REDACTED}
+flag{protect_your_git}
```


The command ```git show``` shows what is added/deleted into the files. Here we only have one file so that's easy to see.

### References:

Really good articles that helped me:

* [Git from the bottom up - Book Open Source !](https://jwiegley.github.io/git-from-the-bottom-up/)
* [Why you should not expose git](https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/)
* [the Git Bible - Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

