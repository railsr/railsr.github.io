---
layout: post
title: Oauth-facebook with devise
category: posts
---
Quick notes on how to add facebook authentication to rails app.

{% highlight ruby %}

#Gemfile

gem 'devise'
gem 'omniauth-facebook'

{% endhighlight %}


{% highlight ruby %}
#user.rb

devise :omniauthable, :omniauth_providers => [:facebook]

def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.provider = auth.provider
      user.uid = auth.uid
      user.email = auth.info.email
      user.username = auth.info.name #gives full user name
      user.password = Devise.friendly_token[0,20]
      user.skip_confirmation!
      user.save
    end
end
{% endhighlight %}

{% highlight ruby %}
#devise.rb

config.omniauth :facebook, ENV['facebook_key'], ENV['facebook_secret'],
 scope: 'email,public_profile', info_fields: 'email, first_name, last_name'
{% endhighlight %}

{% highlight ruby %}
#callbacks_controller.rb

class CallbacksController < ApplicationController
  def facebook
    @user = User.from_omniauth(request.env["omniauth.auth"])
    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication
      flash[:notice] = "Logged in as #{@user.username}"      
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
{% endhighlight %}


{% highlight ruby %}
#routes.rb

devise_for :users, controllers: { omniauth_callbacks: "callbacks" }
{% endhighlight %}

{% highlight erb %}

<%#_links.html.erb%>

<%- if devise_mapping.omniauthable? %>
  <%- resource_class.omniauth_providers.each do |provider| %>
    <%= link_to omniauth_authorize_path(resource_name, provider), class: "btn btn-blue btn-block" do %>
         <i class="fa fa-facebook"></i> Log in with <%= "#{provider.to_s.titleize}" %>
    <% end %><br />
  <% end -%>
<% end -%>
{% endhighlight %}
