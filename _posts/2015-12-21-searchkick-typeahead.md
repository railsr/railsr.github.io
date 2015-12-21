---
title: "Searchkick with typeahead"
tags: [notes, rails]
---

Adding searchkick with typeahead

![Imgur](http://i.imgur.com/bq95WFR.png)


[searchkick](https://github.com/ankane/searchkick)

[typeahead.js](https://github.com/twitter/typeahead.js/)

typeahead.bundle.min.js


{% highlight ruby %}
#build.rb

searchkick autocomplete: ['title'] 
{% endhighlight %}

{% highlight ruby %}
#builds_controller.rb

  def index
    if params[:query].present?
      @builds = Build.published.search(params[:query], page: params[:page], per_page: 20)
    else
      @builds = Build.get_builds(params[:b_type]).page params[:page]
    end
  end

  def autocomplete
    render json: Build.search(params[:query], autocomplete: false, limit: 10)#(&:title) 
    #had to comment out (&:title) because the search autocomplete displayed the full hash of object.
    #displayKey used insted(see below)
  end
{% endhighlight %}

{% highlight ruby %}
#form

  <%= form_tag builds_path, method: :get, class:"input-group" do %>
    <%= text_field_tag :query, params[:query], class: "form-control typeahead", id: "build_search"%>
    <span class="input-group-btn">
      <%= button_tag "Go!", class:"btn btn-primary" %>
    </span>
  <% end %>



{% endhighlight %}

{% highlight bash %}

#application.js

jQuery(function($) {  
  var $vartypeahead = $('#build_search');
  var engine = new Bloodhound({
    remote: { 
        url: "/builds/autocomplete?query=%QUERY",
        wildcard: "%QUERY"
    },
    datumTokenizer: function(d) { return d;},
    queryTokenizer: function(d) { return d;}
  });
  engine.initialize();

  $vartypeahead.typeahead({
    "minLength": 3,
    "highlight": true
  },
  {  
    displayKey: function(build){ return build.name+': '+build.spec},
    #check def autocomplete above for explanation
    "source": engine.ttAdapter()
    }); 
});
 
{% endhighlight %}



