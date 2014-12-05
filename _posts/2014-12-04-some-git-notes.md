---
title: "Some git notes"
tags: [git]
---

Basically to contribute to a git repo you just have to follow these steps:<br/>
1) Fork a repo. 2) Clone it. 3) Create a branch. 4) Make some changes. 5) Commit. 6) Push.
{% highlight Bash shell scripts %}
git clone https://github.com/your-username/forked-repo-name
git checkout -b new-branch-name
git commit -a -m "commit message"
git push origin new-branch-name
{% endhighlight %}

When you need to reset back to previously commited changes do the following:
{% highlight Bash  %}
git reset --hard 9cf35dd8
git push --force origin master
{% endhighlight %}
Destroys all local modifications, uncommited work. And will also reset the remote branch.

If you don't want to break branch history but want to revert commited changes, use "revert".
{% highlight Bash  %}
git revert 9cf35dd8
{% endhighlight %}

If you want to keep your work before resetting use "git stash". 
Then after reset you can drop saved changes with "git drop" or bring them back with "git pop".

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
{% endhighlight %}
