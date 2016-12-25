These instructions make the following assumptions:
- You've just finished using the AWS web console to launch an EC2 server with a 64 bit version of Amazon Linux AMI.

- You've configured your security groups to allow for incoming SSH connections from your local IP.

- You've configured a domain name (or subdomain) to point to your server's IP address.

### Getting started

1. SSH into your new server with the keys you should have received after launching an instance:

	```
	ssh -i /path/to/key.pem ec2-user@domain.com
	```

1. Update your system:

	```
	sudo yum update
	```

1. Install RVM (Ruby Version Manager):

	```
	gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
	
	\curl -sSL https://get.rvm.io | bash -s stable
	```

1. Begin using RVM in current session:
	
	```
	source /home/ec2-user/.rvm/scripts/rvm
	```

1. Install Ruby

	```
	rvm install ruby
	```
	
	This should install the latest version of ruby (2.3 at the time of this writing.) 
	
	*Note that at least Ruby 2.2.2 is required for Rails 5.*

1. Use Ruby

	```
	rvm use ruby
	```

1. Install Bundler:
	
	```
	gem install bundler --no-ri --no-rdoc
	```

1. Install MySQL (optional; you can also use a hosted db through Amazon RDS, which is recommended):
	
	```
	sudo yum install mysql55-server
	sudo service mysqld start
	sudo mysql_secure_installation
	sudo yum install mysql-devel
	sudo chkconfig mysqld on
	```

	Create a database:

	```
	mysql -u root -p
	> create database items;
	> quit;
	```

1. Install Passenger:

	```
	sudo yum install rubygems
	gem install rubygems-update --no-rdoc --no-ri
	update_rubygems
	gem install passenger --no-rdoc --no-ri
	```

1. Remove system Nginx installation if installed (you'll use Passenger's instead):

	```
	sudo yum remove nginx
	sudo rm -rf /etc/nginx
	```

1. Configure Passenger:

	```
	sudo chmod o+x "/home/ec2-user"
	sudo yum install libcurl-devel
	rvmsudo passenger-install-nginx-module
	rvmsudo passenger-config validate-install
	```
	
1. Install Git:
	
	```
	sudo yum install git
	```

1. Set up HTTPS/SSL for your server (free using LetsEncrypt) (required if using the secure client on https://app.neeto.io):

	```
	cd /opt
	git clone https://github.com/letsencrypt/letsencrypt
	cd letsencrypt
	```

	Run the setup wizard:

	```
	./letsencrypt-auto certonly --standalone --debug
	```

	Note the location of the certificates, typically `/etc/letsencrypt/live/domain.com/fullchain.pem`


1. Configure Nginx:
	
	```
	sudo vim /opt/nginx/conf/nginx.conf
	```
	
	Add this to the bottom of the file, *inside* the last curly brace:
	
	```
	server {
	    listen 443 ssl default_server;
	    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
	    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
	    server_name domain.com;
	    passenger_enabled on;
	    passenger_app_env production;
	    root /home/ec2-user/standard-file/public;
	  }
	```


1. Make sure you are in your home directory and clone the Standard File [ruby-server](https://github.com/standardfile/ruby-server) project:
	
	```
	cd ~
  	git clone https://github.com/standardfile/ruby-server.git
	cd ruby-server
	```

1. Setup project:
	```
	bundle install
	rails db:migrate
	rails assets:precompile
	```

1. Create a .env file for your environment variables. The Rails app will automatically load these when it starts.

	```
	vim .env
	```

	Insert:
	
	```
	RAILS_ENV=production
	SECRET_KEY_BASE=use "bundle exec rake secret"
	
	DB_HOST=localhost
	DB_PORT=3306
	DB_DATABASE=items
	DB_USERNAME=root
	DB_PASSWORD=
	
	SMTP_HOST=your-smtp-host
	SMTP_PORT=25
	SMTP_USERNAME=username
	SMTP_PASSWORD=password
		
	PRESENTATION_HOST=https://domain.com
        SINGLE_USER_MODE=false
        SALT_PSEUDO_NONCE=41ff3804086046cdce2836909535c648
	```
	
	Generate your own encryption keys (EKs) for the above variables using Ruby:
	`Digest::SHA256.hexdigest(SecureRandom.random_bytes(32))`
	
1. Start Nginx:
	
	```
	sudo /opt/nginx/sbin/nginx
	```

	Tip: you will need to restart Nginx whenever you make changes to your environment variables or the Nginx configuration. To do so:
	1. Stop Nginx (if it's running): `sudo kill $(cat /opt/nginx/logs/nginx.pid)`
	2. Start Nginx: `sudo /opt/nginx/sbin/nginx`

1. You're done!

## Using your new server
You can immediately start using your new server by using the Standard Notes app at https://app.standardnotes.org.

In the account menu, enter the address of your new server and press Change Server:

![neeto-account-menu](http://imgur.com/Pre6ffL.png)

Then, register for a new account, and begin using your private new secure Standard File server!