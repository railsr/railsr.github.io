---
layout: post
title: Deploying rails app to DO ubuntu droplet with Nginx/Passenger/Mina
category: posts
---

$10 droplet or higher is preferable.

---

##Setup ubuntu server

Ubuntu 14.4 was used

[Setup ubuntu server](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)

{% highlight bash %}
ssh root@SERVER_IP_ADDRESS
adduser demo
gpasswd -a demo sudo

ssh-keygen
cat ~/.ssh/id_rsa.pub
#copy the output

su - demo
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
#paste public key and save
chmod 600 .ssh/authorized_keys
exit

nano /etc/ssh/sshd_config
# edit set PermitRootLogin from yes to no
PermitRootLogin no
service ssh restart

{% endhighlight %}

###Firewall

{% highlight bash %}
sudo ufw allow ssh
sudo ufw allow 4444/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 25/tcp
sudo ufw show added
sudo ufw enable
{% endhighlight %}

###Timezones

{% highlight bash %}
sudo dpkg-reconfigure tzdata
sudo apt-get update
sudo apt-get install ntp
{% endhighlight %}


###Swap File

{% highlight bash %}
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
{% endhighlight %}

---

[Phusion passenger library [guide]](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/digital_ocean/nginx/oss/install_language_runtime.html)

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git-core
sudo apt-get install nodejs
sudo apt-get install -y curl gnupg build-essential
{% endhighlight %}

##rvm

{% highlight bash %}
curl -L https://get.rvm.io | bash -s stable
# will probably give you an error. Just follow instructions.
gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
source /etc/profile.d/rvm.sh
# then relogin to shell
{% endhighlight %}

{% highlight bash %}
echo '[[ -s "/etc/profile.d/rvm.sh" ]] && . "/etc/profile.d/rvm.sh" # Load RVM function' >> ~/.bashrc
rvm install 2.2.2
rvm use 2.2.2 --default
echo "gem: --no-ri --no-rdoc" > ~/.gemrc
{% endhighlight %}

##Nginx and Passenger

{% highlight bash %}
# passenger
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 561F9B9CAC40B2F7
sudo apt-get install apt-transport-https ca-certificates
sudo sh -c 'echo "deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main" >> /etc/apt/sources.list.d/passenger.list'
sudo chown root: /etc/apt/sources.list.d/passenger.list
sudo chmod 600 /etc/apt/sources.list.d/passenger.list
sudo apt-get update

# nginx
sudo apt-get install nginx-full passenger
{% endhighlight %}


{% highlight bash %}
passenger-config --ruby-command
# copy ruby path for nginx
# To use in Nginx : passenger_ruby /usr/local/rvm/gems/ruby-2.2.2/wrappers/ruby

# then update it in /etc/nginx/nginx.conf

# uncomments this lines and update your ruby path
# passenger_root /some-filename/locations.ini;
# passenger_ruby /usr/bin/passenger_free_ruby;

sudo nano /etc/nginx/nginx.conf

sudo service nginx restart

{% endhighlight %}


##Validating installation
[Passenger library](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/digital_ocean/nginx/oss/trusty/install_passenger.html#step-3:-check-installation)

To validate installations of passenger, run the following command:

{% highlight bash %}
sudo passenger-config validate-install
{% endhighlight %}



You can also check passenger mem stats
{% highlight bash %}
sudo passenger-memory-stats
{% endhighlight %}


##PostgreSQL

[[do article]](https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-ruby-on-rails-application-on-ubuntu-14-04)

{% highlight bash %}
sudo apt-get install postgresql postgresql-contrib libpq-dev
sudo -u postgres createuser -s pguser
sudo -u postgres psql

# psql

\password pguser
# enter password for this user
\q

{% endhighlight %}

You can also do it differently [[cyberciti]](http://www.cyberciti.biz/faq/howto-add-postgresql-user-account/)

{% highlight bash %}
sudo su - postgres
psql
# psql

CREATE USER pguser WITH PASSWORD 'myPassword';
CREATE DATABASE database_name; # name of db in your app
GRANT ALL PRIVILEGES ON DATABASE database_name to pguser;
\q
{% endhighlight %}

##Mina

A lot of info can be found in the repo of the gem.
[mina github](https://github.com/mina-deploy/mina)


{% highlight bash %}
mina init # will create a config/deploy.rb file
# open it and configure

{% endhighlight %}

you can use [this one](https://gist.github.com/railsr/efbac95f70f0f3837e86) just make some basic edits: domain, deploy_to....



Then make a directory for your app on your server

{% highlight bash %}
sudo mkdir -p /var/www/myapp
sudo chown -R myappuser /var/www/myapp
{% endhighlight %}

Now you can run `mina setup` or `mina setup --verbose` to see logs

After this go to your server and `cd` to `shared/config` and update your `yml` files.
In my case, they are: `database.yml`, `secrets.yml`, `application.yml`

Then run `mina deploy` or `mina deploy --verbose`

##Configure Nginx

If you want to host a few apps on your server you can create a separate nginx config file for your app. Or just use the default. `/etc/nginx/sites-enabled/default`


Inside `/etc/nginx/sites-enabled/` create a config file of your website `myapp.conf`

{% highlight Nginx config files %}
server {
    listen 80;
    listen [::]:80;
    server_name domain_name.com;
    # Tell Nginx and Passenger where your app's 'public' directory is
    root /var/www/myapp/current/public;
    # Turn on Passenger
    passenger_enabled on;
    passenger_ruby /home/username/.rvm/gems/ruby-2.2.2/wrappers/ruby;
}
{% endhighlight %}    



Then run:

{% highlight bash %}
sudo service nginx restart
{% endhighlight %}

Now you should have your app running on your server ip address.
