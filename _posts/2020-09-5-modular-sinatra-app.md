---
layout: post
title: Modular sinatra app
category: posts
---

I recently saw a post on reddit(can't find it) where a guy posted a link to his sinatra [jobs-tracker app](https://github.com/Allenfp/job-app-tracker). This kinda made me wanna try [Sinatra](http://sinatrarb.com) and I really liked it. It seems perfect for small/hobby or even average projects. 

I'm trying to get back into wedev and after a long hiatus, the first thing I noticed about Rails is: creating new projects takes too much time. A couple years ago it was like 10-20 secs. And now, on my 2015 rmbp, it takes over a minute or two. With Sinatra, everything is really fast and you don't get stuff that you don't nesseserily need, included in your project.

So, this jobs-tracker app I talked earlier inspired me to build this **[Issue Tracker](http://github.com/kirqe/puuci)** kinda thing where you can create projects, open issues on projects and leave comment for issues.

Modular sinatra app structure is simlar to the basic Rails project. You might decide to structure it differently though - there're no strict requirements/conventions.

Here's how I structured my app:

![modular sinatra app](/public/post_files/modsin.png)

The whole app is run through `config.ru` file. `.ru` is a rack up file.

In the `config.ru` you require configuration files, classes and everything that should be loaded as the app starts.

```
require_relative './config/environment'
require_relative './lib/sinatra/helpers'

use RegistrationsController
use UsersController
use ProjectsController

run ApplicationController
```

After creating a new controller, you specify it in this file, using `use` keyword. And to run the whole app - `run`. In this case, `ApplicationController` is like the the starting point of your project. Just like `app.rb` that you have in a single file sinatra app, that you run through `ruby app.rb`

For developement purposes you might want to use something like [shotgun](https://github.com/rtomayko/shotgun). It feels slow because of code reload that happens as you make changes to the files. Great for developement purposes. 

To run the app you just execute `shotgun` from the app directory and it should be available at `localhost:9393` unless you specify port with `--port=9292`

Another way to run your app is `rackup config.ru`. This time the app should feel snappy but you're not getting code reload.

```
rspec --init
rspec spec/
```


## Link to delete and sending POST requests

Sinatra doesn't have all the helpers that you get with rails. Which means you can't just do the following and have and have an item deleted with confirmation box.

```slim
 = link_to 'Delete Item', data: {confirm:"Are you sure?"}
 ```

There're at least 2 solutions to this problem that I found: 

(1) create a form that sends a POST request like this one:
 
 ```slim
 form action="/items/#{item.id}" method="POST" id="del_#{item.id}"
  div style="margin:0;padding:0"
    input name="_method" type="hidden" value="delete"
  a href='javascript:void()' onclick='document.getElementById("del_#{item.id}").submit();' DELETE ITEM
```

(2) or just install `jquery-ujs`
  
```slim 
a href="/item/#{item.id}" data-confirm="Are you sure?" data-method="delete" Delete item
```

You have to add `use Rack::MethodOverride` to app.rb or config.ru and have something like this in your controller:
>"Reminder: Remember to add the use Rack::MethodOverride to your config.ru file so that your app will know how to handle patch and delete requests!"

```ruby
  delete '/items/:id' do |id| # <- notice the block
    @item = Item.find(params[:id])
    @item.destroy
    redirect to '/items'
  end
```

## Authentication
At first I wanted to use `bcrypt` and it's `authenticate` method but then decided to go with what's used by lots of projects: Warden.
[This gist](https://gist.github.com/lukesutton/107966) covers pretty much everything you need to know for setup.

## Authorizing requests

Checking the docs turns out to be a really good idea.
At first I was trying to play with `before`  filter combining it with redirects and `warden.authenticate!` It didn't look good and there was not much information regarding authorization in google.

Then after checking [the docs](https://github.com/sinatra/sinatra#conditions) I found that sinatra comes with conditions. So authorization can look like this:

```ruby
  # application_controller.rb
  set(:auth) do |*roles|
    condition do
      if logged_in?
        unless roles.any? {|role| current_user.in_role? role }
          flash :error, "You're not authorised to access this page"
          redirect "/"
        end          
      else
        env['warden'].authenticate!
      end
    end
  end    
```

```ruby
  # models/user.rb
  def in_role?(role)
    self.role == role
  end
```

```ruby
  get '/projects', :auth => ["user", "admin"] do
    @projects = Project.all
    slim :'projects/index'
  end
```

But how to authorize resources that belong to certain users? 
For instance, when someone wants to update a resource(project or issue) that was created by them? 
The code above only checks for user roles.

There're a couple of old gems suited for this task but I went with a simple helper `authorize(resource)`

```ruby
def authorize(resource)
  if resource.class.name == "User"
    unless resource == current_user
      flash :error, "Not authorized"
      redirect "/"
    end
  elsif resource.class.name == "Upload"
    unless resource.uploadable.user == current_user
      flash :error, "Not authorized"
      redirect "/"
    end
  else
    unless current_user.is_admin? || resource.user == current_user
      flash :error, "Not authorized"
      redirect "/"
    end
  end
end
```

Now when I need to check if resouce belongs to a user I do the following:

```ruby
class ProjectsController
    patch '/projects/:id' do
        @project = Project.find(params[:id])
        authorize(@project)
        ....
    end
end
```

I was a big fan of `haml` and was skeptical about `slim` but after getting used to it there's no way back :)

### Helpful resources

- [Sinatra github repo has lot of helpful information with examples](https://github.com/sinatra/sinatra#sinatra)
- [Warden gist example](https://gist.github.com/lukesutton/107966)
- [Litecable example](https://github.com/palkan/litecable/tree/master/examples/sinatra)
- [Fileupload](https://github.com/tbuehlmann/sinatra-fileupload)

