---
title: "Some git notes"
tags: [git]
---

Basics:<br/>
{% highlight Bash shell scripts %}
git clone https://github.com/your-username/forked-repo-name
git checkout -b new-branch-name
git commit -a -m "commit message"
git push origin new-branch-name
{% endhighlight %}
then pull request.

"git reset" will reset back to previously committed/pushed changes. It rewrites commit history, destroys all local modifications and uncommitted work. And will also reset the remote branch.
{% highlight Bash  %}
git reset --hard 9cf35dd8
git push --force origin master
{% endhighlight %}

{% highlight Bash  %}
git reset --hard # removes staged and working directory changes
git clean -f -d # remove untracked files
git clean -f -x -d # CAUTION: as above but removes ignored files like config.
{% endhighlight %}

"git revert" allows to revert committed changes.
{% highlight Bash  %}
git revert 9cf35dd8
{% endhighlight %}

"git stash" allows to keep work before resetting/deleting... a branch. After reset/delete... saved changes can be destroyed with "git drop" or applied to new a branch with "git pop".

{% highlight Bash %}
git stash
git reset --hard 9cf35dd8
git drop or pop 
git push --force origin master
{% endhighlight %}

Some other git commands:
{% highlight Bash  %}
git branch -r #shows remote branches
git status #shows changed files before commit
git diff master..origin/master #diff between local and remote br
git log --all --graph --decorate --oneline --abbrev-commit
{% endhighlight %}
