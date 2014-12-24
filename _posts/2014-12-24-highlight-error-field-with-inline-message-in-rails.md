---
title: "Highlight error form field with inline error message in rails."
tags: [ruby on rails]
---

Here's an example for username field of devise signup form. Other fields are almost the same. Just replace the hash key.

{% highlight bash %}
<div class="form-group <%= "has-error" if f.object.errors.include?(:username) %>">
	<%= f.label :username, :class => "col-sm-2 control-label" %>
	<div class="col-sm-4">
	<%= f.text_field :username, autofocus: true, :class => "form-control", :placeholder => "Enter desired username"%>
	</div>
	<span class="help-block"> <%= f.object.errors[:username].join(", ") %></span>
</div>
{% endhighlight %}


Result:<br/>
![Imgur](http://i.imgur.com/Pv1g3UR.png)
