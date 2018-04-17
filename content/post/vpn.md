---
title: "Free VPN server on GCP in 5 minutes"
date: 2018-04-17T09:29:16+03:00
---

As you might know, there are a lot of blocked services in Russia, LinkedIn, Telegram, etc.,
and lately, I start to use LinkedIn more extensively. I tried to use some free VPN services, like [FlyVPN](https://www.flyvpn.com/), [Tunnel Bear](https://www.tunnelbear.com/),
but they all have restrictions and slow. That's why I decided to run my own VPN server on some cloud platform.
I know that you can run droplet in the [digital ocean for only 5$ a month](https://www.digitalocean.com/pricing/)
{{% center %}}
![droplet for 5$](/droplet.png),
{{% /center %}}
the same for [linode](https://www.linode.com/pricing)
{{% center %}}
![linode server](/linode.png)
{{% /center %}}
but on Google Cloud Platform, you can have [always free f1-micro instance](https://cloud.google.com/free/docs/always-free-usage-limits),
so I decided to go that way.

First, we create new `micro` instance in one of the us-east1, us-west1, and us-central1 regions.
I'm using image `Ubuntu 17.10`, we don't need any SSD, just persistent 10Gb disk,
we need to allow HTTP and HTTPS traffic, our VPN server will be available over HTTPs
and we need HTTP for LetsEncrypt verification.
{{% center %}}
![create new instance](/create_instance.png)
{{% /center %}}
In the `VPC network->Firewall rules` we need to add port for our VPN server, I'm using port `18450` you can use another.
{{% center %}}
![add firewall rule](/firewall_rule.png)
{{% /center %}}
Next, we need to add the tag from firewall rule to our new instance. So go to our newly created instance and press `Edit`:
{{% center %}}
![add tag](/add_tag.png)
{{% /center %}}
while we editing instance setting we need also to add external static IP:
under `Network interfaces`
{{% center %}}
![Network interfaces](/network.png)
{{% /center %}}
create new static IP
{{% center %}}
![add static IP](/static_ip.png)
{{% /center %}}

If you like me and all your DNS record already in `Cloud DNS` you can add new DNS record for this IP under `Network services->Cloud DNS`:
{{% center %}}
![add DNS name](/dns.png)
{{% /center %}}
We will need this DNS name for LetsEncrypt verification.

OK, our instance is ready, now we need to install VPN server.
Connect to the instance through SSH web-based client:
{{% center %}}
![SSH](/ssh.png)
{{% /center %}}

We will be using [PRITUNL](https://pritunl.com/) because it has excellent webUI.
Official instruction can be found [here](https://docs.pritunl.com/docs/installation), but it's very simple, so I'll show all steps here:

just run following command

{{< highlight bash >}}
sudo tee -a /etc/apt/sources.list.d/pritunl.list << EOF
deb http://repo.pritunl.com/stable/apt artful main
EOF

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
sudo apt-get update
sudo apt-get --assume-yes install pritunl mongodb-server
sudo systemctl start pritunl mongodb
sudo systemctl enable pritunl mongodb
{{< /highlight >}}

That's all, now you can go to https://vpn.youdomains.com
{{% center %}}
![pritunl setupkey](/pritunl_setup_key.png)
{{% /center %}}
To get the setup key run the command `pritunl setup-key` this will return the setup key.
By default, the MongoDB uri will be filled with the uri for the localhost MongoDB server.
This should be left as it is when the MongoDB server is running on the same server as the Pritunl instance. 

After that, you already can login with username and password - `pritunl`

Next, on the `Initial Setup` screen, you need to set `Username`, `New Password`,
`Public Address` and set `LetsEncrypt Domain` to your newly created domain name. 
{{% center %}}
![initial setup](/initial_setup.png)
{{% /center %}}
Now we just need to add organization
{{% center %}}
![add organization](/create_org.png)
{{% /center %}}
add new user to this organization
{{% center %}}
![add user](/create_user.png)
{{% /center %}}
add new server
{{% center %}}
![add server](/add_server.png)
{{% /center %}}
don't forget to set port the same as in `Firewall rule`

attach server to the organization
{{% center %}}
![add server](/attach_org.png)
{{% /center %}}
Then click Start Server to start the vpn server.

After the server has been created the user profile can be downloaded on the
`Users` page by clicking the download button or profile links button on the right side of a user.
The profile can then be imported into the `Pritunl` client or any other `OpenVPN` client.

That's all, your VPN server is ready.

I use [pritunl client](https://client.pritunl.com/) on macos and openvpn on [android](https://play.google.com/store/apps/details?id=net.openvpn.openvpn) and on [iOS](https://itunes.apple.com/ru/app/openvpn-connect/id590379981?mt=8)
