---
layout: post
title:  "Local Port Forwarding via SSH"
date:   2021-06-11 20:31:00 +0000
categories: ssh
---

{% highlight shell %}
apt install -y apache
{% endhighlight %}


`ssh -L [LOCAL_IP:]LOCAL_PORT:DESTINATION:DESTINATION_PORT [USER@]SSH_SERVER`

{% highlight shell %}
ssh -L localhost:80:192.168.0.2:80 user@{remote_ssh_public_ip}
{% endhighlight %}

