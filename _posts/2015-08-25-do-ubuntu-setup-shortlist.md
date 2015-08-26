---
title: "Digital Ocean ubuntu setup commands short list"
tags: [notes, deploy, rails]
---

A list of steps on how to make ubuntu initial setup on DO (without description).

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
