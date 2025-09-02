




# How I Built My Own Forward Proxy Server on AWS EC2 with Squid
![[proxy-server-1.webp]]


When I first started learning web scraping, I came across a serious problem: **IP blocking**.  
Websites don’t like when a single IP makes too many requests, and once that happens, they block you.

So I began exploring: _How do people actually avoid getting their IPs blocked?_  
Turns out, there are a few ways — some work, some don’t.

## I found these ways to avoid IP blocking.


#### 1. ISP Reconnection

Most Internet Service Provider give you a **dynamic IP**, meaning it changes every time you restart your router.  
So, in theory, I could just keep turning my router off and on.

But let’s be honest — I can’t sit there flipping the router all day just to scrape data. 😭

---
#### 2. Using VPNs

I tried VPNs next. But here’s the catch:
- Free VPNs are basically a trap — they’re slow, inject ads, and even leak your data.
- Paid VPNs are better, but some still keep hidden logs despite claiming _“no logs”_.

Not ideal if you actually care about privacy or reliability.

---

#### 3. Proxy Servers

This is where things got interesting.
A **proxy server** is basically another computer that sits between you and the internet:
- You send your request → proxy forwards it → website responds → proxy sends it back.
- Your real IP stays hidden.

There are two main options:
- **Free Proxies** → unreliable, super slow, often dead.
- **Paid Proxies** → reliable, fast, but cost money.

Companies like **Oxylabs** or **Proxy-Cheap** sell them starting around $3/month, with pools of rotating IPs.
But as a broke college student 😭, I didn't had money to pay for them.  
So I asked myself: _Why not just build my own proxy server?_

---


## Start with the fundaments : 

## First let's understand what is Proxy Server ?

- A **proxy server** is like a middleman between you and the internet.
- Instead of talking directly to a website, your request first goes to the proxy.
- The proxy then forwards your request, gets the response, and sends it back to you.
- This hides your real **IP address** from the website.
- It can be used for **privacy**, **bypassing restrictions**, or **improving performance**.
- Companies use proxies to **filter traffic** or block unwanted sites.
- In short: a proxy is a **mask** for your internet identity and activity.


## So, What's Squid? 🤔

- Squid is basically a **proxy caching server**.
- It can run in **forward proxy mode** (your requests → proxy → internet) or **reverse proxy mode** (clients → proxy → backend servers).
- For my use case (web scraping, testing with different IPs), I wanted the **forward proxy** setup.
So Squid was a perfect fit.


## ⚡ Why AWS EC2 + Squid?

There are tons of proxy services out there, but setting it up on EC2 gave me a few advantages:

- A **dedicated IP** that I fully control.
- **Access control** with user/password auth.
- Easy to **scale** if I need more proxies later — just launch more EC2 machines 😎.

And honestly, it just felt cool to say:  
_"Yeah, I run my own proxy servers on AWS."_ 😎


## Let's Build it :

### Step 1: Launch an EC2 Instance

Heres how to start:
- Chose **Ubuntu 22.04** (Amazon Linux 2 works too).
- Picked `t3.micro` (free tier) since we didn’t need much power.
- Configured the **security group** to allow:
    - SSH (`22`) → my IP only or `0.0.0.0/0` for now.
    - Custom TCP (`3128`, Squid’s default port) → my IP only (or `0.0.0.0/0` ).
⚠️ Pro tip: Don’t allow access from anywhere :  `0.0.0.0/0`
![[Pasted image 20250902000245.png]]


### step 2 : SSH to the EC2 instance : 

```bash
ssh -i "your-pem-file.pem" ubuntu@your-public-ip-here.ap-south-1.compute.amazonaws.com
```

- replace `your-pem-file.pem` with your pem file name and make sure you have given executable permission to it using `chmod 400 "your-pem-file.pem"`


## step 3 : Insall squid : 

```bash
sudo apt update -y
sudo apt install -y squid apache2-utils
```
That got Squid running with a default config.

### step 4 : Configure Authentication

1. create a password file : 

```bash
cd /etc/squid
sudo htpasswd -c ./passwords myuser 
```
- replace `myuser` with your desire username
- set password, we will use them for authentication.

2. Edit the config:
```bash
sudo vim conf.d/debian.conf
```

Add these 4 lines at the bottom:
```bash
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm proxy

acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```
- Now your proxy server is password protected.

3. Restart the squid:
```bash
sudo systemctl restart squid
sudo systemctl status squid
```
![[Pasted image 20250901234553.png]]


### step 5: Time to Use the Proxy!

This is the moment of truth. The proxy is ready to use with the following format: `http://username:password@your_ec2_public_ip:3128`

To test it from my own terminal, run a quick `curl` command. This command asks the website `api.ipify.org` (which just tells you your public IP) for its content, but it routes the request through my new proxy.
```bash
curl -x http://myuser:mypassword@EC2_PUBLIC_IP:3128 https://api.ipify.org
```
replace
- `myuser` with your username 
- `mypassword` with your password
- `EC2_PUBLIC_IP` with the public IP of your EC2 instance.

output : If it returns your EC2’s IP → congratulations, your proxy is alive 🎉.

You can check the logs of the proxy from `/var/log/squid/access.log`

```bash
sudo cat /var/log/squid/access.log
```

### step 6 : configure your proxy in web browser : 

I am using Firefox, but setting are similar in other browsers also
- Open **Settings** → **Network Settings** → _Configure how Firefox connects to the Internet_.
- Select **Manual proxy configuration**.
- Enter your HTTP/HTTPS proxy: (public IPv4 address)
- If the proxy requires authentication, Firefox will prompt for **username/password** when connecting.
![[Pasted image 20250901235343.png]]



### ## A Final Word on Security (Please Read!)  🙏

Building this was fun, but it's crucial to be responsible.

- **NEVER run an open proxy.** Always secure it with password authentication or by whitelisting your IP in the EC2 security group. Open proxies get abused very quickly.
    
- **Use an Elastic IP.** If you stop and start your EC2 instance, its public IP will change. By attaching a free AWS Elastic IP, your proxy's address stays the same.
    
- **Check the logs.** Squid logs all activity to `/var/log/squid/access.log`. It's good practice to keep an eye on it.

## Hope this guide helps you build your own!, if you find this interesting then please upvote this 🙂

![[Pasted image 20250902000359.jpg]]


# sdf