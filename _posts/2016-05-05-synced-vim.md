---
layout: single
title: Synced Vim across different locations
category: howto
tags: [vim, git, github]
---
My favorite editor is [vim](https://vim.org). How do I manage to have all
configurations, plugins etc. synced between different accounts/workplaces?

Simply using [git](https://git-scm.com/) and [GitHub](https://github.com). I
have repository where I store all my configs, plugins and scripts on GitHub.
This repository is checked out in each location where I'm working. Usually it's
in `~/src/vim` and then symlinks from `~/src/vim/.vimrc` to `~/.vimrc` and
`~/src/vim/.vim` to `~/.vim`.

```bash
$ ls -la .vimrc .vim

lrwxr-xr-x  1 karl  staff     12 Apr  9 13:03 .vim -> src/vim/.vim
lrwxr-xr-x  1 karl  staff     14 Apr  9 13:02 .vimrc -> src/vim/.vimrc
```

Each time there is some change, easily commit the change and push the changes.
On any other machine just pull changes. Easy!

Keeping plugins updated
-----------------------

This solves issues with having vimrc and vim directory synced. But what about
plugins like [fugitive](https://github.com/tpope/vim-fugitive) which are hosted
on GitHub and they have active development?
[git-subtree](https://git.kernel.org/cgit/git/git.git/tree/contrib/subtree/git-subtree.txt)
is the remedy! Simply create subtree for each plugin in `.vim/bundle` directory.
This expects that you are using
[pathogen](https://github.com/tpope/vim-pathogen). 


* install pathogen and follow instruction for directory structure

* find your best plugin url `https://github.com/tpope/vim-fugitive`

* in root of your vim repo execute `git subtree`

```bash
$ git subtree add --prefix .vim/bundle/vim-fugitive --squash https://github.com/tpope/vim-fugitive master

git fetch https://github.com/tpope/vim-fugitive master
From https://github.com/tpope/vim-fugitive
 * branch            master     -> FETCH_HEAD
Added dir '.vim/bundle/vim-fugitive'
```

* `git log` returns something similar

```bash
Commit 31ecc97fe70092a99ddc913d3615e3a0dcd9d87c
Merge: a20490b 4cd5c7b
Author: Karel Rank <karel.rank@gmail.com>
Date:   Tue May 3 22:00:40 2016 -0400

   Merge commit '4cd5c7b009497155b18ee4b574207838f73c13b0' as '.vim/bundle/vim-fugitive'

commit 4cd5c7b009497155b18ee4b574207838f73c13b0
Author: Karel Rank <karel.rank@gmail.com>
Date:   Tue May 3 22:00:40 2016 -0400

   Squashed '.vim/bundle/vim-fugitive/' content from commit bdd2168

   git-subtree-dir: .vim/bundle/vim-fugitive
   git-subtree-split: bdd216827ae53cdf70d933bb30762da9bf42cad4
```

* when plugin is updated run `git subtree pull`

```bash
$ git subtree pull --prefix .vim/bundle/vim-fugitive --squash https://github.com/tpope/vim-fugitive master

From https://github.com/tpope/vim-fugitive
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 .vim/bundle/vim-fugitive/plugin/fugitive.vim | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
```

Unfortunately you have to remember url of the source repo. But patch to store
repo into commit message was submited
<http://article.gmane.org/gmane.comp.version-control.git/288606>
