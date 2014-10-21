---
title: "How to contribute to a git repo"
tags: [git]
---


So, here're the steps of how to contribute to someone's git repo:

1) Fork a repo. 2) Clone it. 3) Create a branch. 4) Make some changes. 5) Commit. 6) Push.

{% highlight Bash shell scripts %}
git clone https://github.com/your-username/forked-repo-name
git checkout -b new-branch-name
git commit -a -m "commit message"
git push origin new-branch-name
{% endhighlight %}


Some other commands:

{% highlight Bash shell scripts %}
git branch -r 
git status
git diff
{% endhighlight %}




