# Ansible playbook for setting up a tilde.club like server

This repo contains an ansible playbook that will setup a [tilde.club](http://tilde.club) like server.  Basically it transforms a CentOS 7 box into a server that offers: shell accounts, static ~username webhosting, and a chance to network with other geeks via private email and IRC systems.   

This was written with security in mind.  
- selinux enforcing
- properly configured firewall
- automatic updates
- resource limits on users via limits.conf
- logwatch log reporting
- disk usage reports
- auditd logging every command users run
- aide intrustion detection
- ssh tunneling disabled, only key based auth allowed
- fail2ban configured for ssh
- only outgoing web and dns traffic allowed
- tons of apache modules disabled
- postfix and ngircd locked to localhost

## Standing up a server

1. Clone this repo and install ansible on your machine
2. Stand up a standard CentOS 7 VPS.  Amazon linux not tested.  OpenVZ containers not tested.  I tested against an hourly instance from ArubaCloud.
3. Point the @, A, www and any other DNS records you want at your VPSes IP. 
4. Login to your VPS as root enable selinux and create an account for admin work giving it full sudo rights and upload your ssh key to it
    
	    useradd sysop 
	    visudo
	    sysop ALL=(ALL) NOPASSWD: ALL
	    vi /etc/sysconfig/selinux #set SELINUX=enforcing SELINUXTYPE=targeted and reboot if its not like that
	    su - sysop
	    mkdir ~/.ssh
	    chmod 700 ~/.ssh
	    vi ~/.ssh/authorized_keys
	    chmod 600 ~/.ssh/authorized_keys


5. Populate users.csv with username,sshkey entries for your users. Do NOT include the admin user you manually created above, but feel free to create another normal user account for yourself here.  
6. Customize the following to your liking

- playbook.yml vars:

    - fqdn #must equal the top level DNS name for your site.  IE: tilde.club
    - sysop #uid of the admin user you created above. will be IRC OP and will receive cron emails
    - ircoppass #IRC OP password
    - letsencrypt #keep false until I implement it

- replace 127.0.0.1 in inventory with the IP of your VPS
- templates/general/limits.conf.j2  #resource limits for accounts
- templates/httpd/index.html.j2 #your main website
- templates/ssh/motd.j2 #message users see when they ssh in
- files/general/skel/public_html/index.html #intial ~user webpage 

7. Run the playbook like so 

    >ansible-playbook -i inventory -u sysop -b playbook.yml

8. Get proper HTTPS certs from [LetsEncrypt](https://certbot.eff.org/#centosrhel7-other) and update templates/httpd/tilde.conf.j2 to point to them.


## Adding more users
Add their info to users.csv and re-run the playbook.  

## Removing users
Remove their info from users.csv and do a `userdel -r $username` as root.


## End User Usage

#### Web
Use scp, rsync, ssh + vim, Filezilla, etc to place your static content under you public_html directory.  PHP, CGI, .htaccess, and tons of other features are NOT supported.  Go [here](https://www.staticgen.com/) for some cool tools that will still let you create amazing sites.  

#### Mail
You can only email other folks on this server.  You can only receive email from other folks on this server.  You can only check and send email by being sshed into the server.  

Basic alpine config and send/receive test

	alpine
	enter
	S
	C
	Inbox Path
	enter
	Name of Inbox server: #leave blank
	enter
	Folder name to use for INBOX: Mailbox
	enter
	E
	Y
	C
	To: $username@$domain
	Subject: test
	Message: test
	ctrl - x
	Y
	L
	INBOX 
	enter
	test 
	enter
	q
	Y

#### IRC
You can only chat with other people on this server.  You must be sshed in to connect to the IRC server. 

Basic irssi usage

	irssi
	/connect $domain
	/list #find a channel
	/join #chat
	/n #see who is in the channel
	type words to talk
	switch windows with ctrl-n and ctrl-p. close windows with /window close
	private message somebody with /q username
	start a new channel with /join #newchannel
	/quit to quit


