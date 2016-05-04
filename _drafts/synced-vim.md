My favorite editor is [vim](https://vim.org). How do I manage to have all
configurations, plugins etc. synced between different accounts/workplaces?

Simply using [git](https://git-scm.com/) and [GitHub](https://github.com). I
have repository where I store all my configs, plugins and scripts on GitHub.
This repository is checked out in each location where I'm working. Usually it's
in `~/src/vim` and then symlinks from `~/src/vim/.vimrc` to `~/.vimrc` and
`~/src/vim/.vim` to `~/.vim`.

{% highlight bash %}
~ ls -la .vimrc .vim

lrwxr-xr-x  1 karl  staff     12 Apr  9 13:03 .vim -> src/vim/.vim
lrwxr-xr-x  1 karl  staff     14 Apr  9 13:02 .vimrc -> src/vim/.vimrc

{% endhighlight %}

Keeping plugins updated
-----------------------

This solves issues with having vimrc and vim directory synced. But what about
plugins like [fugitive](https://github.com/tpope/vim-fugitive) which are hosted
on GitHub and they have active development?
[git-subtree](https://git.kernel.org/cgit/git/git.git/tree/contrib/subtree/git-subtree.txt)
is the remedy! Simply create subtree for each plugin in `.vim/bundle` directory.
This expects that you are using
[pathogen](https://github.com/tpope/vim-pathogen). 


1. install pathogen and follow instruction for directory structure
2. find your best plugin url `https://github.com/tpope/vim-fugitive`

3. in root of your vim repo execute g
{% highlight bash %}

git subtree pull -P .vim/bundle/vim-fugitive https://github.com/tpope/vim-fugitive master --squash
From https://github.com/tpope/vim-fugitive
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 .vim/bundle/vim-fugitive/plugin/fugitive.vim | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

{% endhighlight %}
Patch to store repo into commit message was submited <http://article.gmane.org/gmane.comp.version-control.git/288606>
