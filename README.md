# LEMPSTACK
LEMP SERVER

In this project, you will implement a similar stack, but with an alternative Web Server – NGINX, which is also very popular and widely used by many websites on the Internet
 
- create a new EC2 Instance of t2.nano family with Ubuntu Server 22.04 LTS (HVM) image.
![screenshot 1](./image2/1%20shot.png)

### step 1: INSTALLING THE NGINX WEB SERVER

In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.
Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:
`sudo apt update`
`sudo apt install nginx`

When prompted, enter Y to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.
To verify that nginx was successfully installed and is running as a service in Ubuntu, run:
`sudo systemctl status nginx`

If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds!

Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is default port that web browsers use to access web pages in the Internet.
As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:

Our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).
![localhost](./image2/local%20host.png)

First, let us try to check how we can access it locally in our Ubuntu shell, run:
`curl http://localhost:80`

Now it is time for us to test how our Nginx server can respond to requests from the Internet.
Open a web browser of your choice and try to access following url
`http://<Public-IP-Address>:80`

If you see following page, then your web server is now correctly installed and accessible through your firewall


## STEP 2 — INSTALLING MYSQL
Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database.

- Again, use ‘apt’ to acquire and install this software:
 `sudo apt install mysql-server -y`

 ![my sql](./image2/sql%20instal.png)

 When the installation is finished, log in to the MySQL console by typing:
 `sudo mysql`
This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:


It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script, you will set a password for the root user, using mysql_native_password as default authentication method

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'`
Exit the MySQL shell with:
`mysql> exit`
Start the interactive script by running:
 `sudo mysql_secure_installation`

 This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. Answer Y for yes, or anything else to continue without enabling. Exit afterwards

 ## STEP 3 – INSTALLING PHP

 You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server

To install these 2 package, run;
`sudo apt install php-fpm php-mysql -y`

## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use project LEMP as an example domain name.
Create the root web directory for your domain as follows:
`sudo mkdir /var/www/projectLEMP`

Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:
`sudo chown -R $USER:$USER /var/www/projectLEMP`
Then, open a new configuration file in Nginx’s sites-available directory using the command-line, nano:

`sudo nano /etc/nginx/sites-available/projectLEMP`

This will create a new blank file. Paste in the following bare-bones configuration


Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:
`sudo nginx -t`	

You shall see following message:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

If any errors are reported, go back to your configuration file to review its contents before continuing.
We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:
`sudo unlink /etc/nginx/sites-enabled/default`
When you are ready, reload Nginx to apply the changes:
`sudo systemctl reload nginx`

Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected
`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`

Now go to your browser and try to open your website URL using IP address:
`http://<Public-IP-Address>:80`	
If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.

## STEP 5 – TESTING PHP WITH NGINX

At this point, your LAMP stack is completely installed and fully operational.

You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.

You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
`sudo nano /var/www/projectLEMP/info.php`
Type or paste the following lines into the new file. This is valid PHP code that will return information about your server


You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
`http://`server_domain_or_IP`/info.php`

You will see a web page containing detailed information about your server:
After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:
`sudo rm /var/www/your_domain/info.php`
You can always regenerate this file if you need it later.


## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH 
In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

We will create a database named example_database and a user named example_user, but you can replace these names with different values.
First, connect to the MySQL console using the root account:
`sudo mysql`

To create a new database, run the following command from your MySQL console:
`mysql> CREATE DATABASE example_database`

Now you can create a new user and grant him full privileges on the database you have just created.

`mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password'`

Now we need to give this user permission over the example_database database:
`mysql> GRANT ALL ON example_database.* TO 'example_user'@'%'`

This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.
Now exit the MySQL shell with:
`mysql> exit`

You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:
`mysql -u example_user -p`

After logging in to the MySQL console, confirm that you have access to the example_database database:
`mysql> SHOW DATABASES`
This will give you the following output:
[scrreen grab]


Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
To confirm that the data was successfully saved to your table, run:
`mysql> SELECT * FROM example_database.todo_list`
You’ll see the following output:

`nano /var/www/projectLEMP/todo_list.php`

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception
Copy this content into your todo_list.php script:

[screengrab]
Save and close the file when you are done editing

You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:
`http://<Public_domain_or_IP>/todo_list.php`

You should see a page like this, showing the content you’ve inserted in your test table:

That means your PHP environment is ready to connect and interact with your MySQL server








