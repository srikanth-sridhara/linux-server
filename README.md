# Linux Server Configuration

Created by Srikanth Sridhara on 11th Oct 2017

This is a deployment of multiple projects on a Linux Server running Ubuntu using [`Amazon Lightsail`](http://amazonlightsail.com). Right now it is used to display two web apps:

1. Musical Instruments app that uses Flask with CRUD operations on Postgresql DB with Oauth2.
2. Neighborhood Map app that is mainly a front end Map search app using Javascript and 3rd party APIs.


### URLs:
The Public IP of the server is: `52.24.122.174` and SSH port is `2200`.

The homepage of the project is: [`http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com`](http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com)

From here, the two web apps can be accessed thus:
* [`http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com/MusicalInstruments`](http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com/MusicalInstruments)
* [`http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com/NeighborhoodMap`](http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com/NeighborhoodMap)


### Steps taken to setup the server:

1. First, I created an account on Lightsail and started a server instance. I `updated` all the packages and then `upgraded` the installed packages using apt-get.

2. Next, I enabled port 2200 for TCP in Lightsail firewall settings. After that, I changed the firewall settings in the server(ufw) to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
<pre>
grader@ip-172-26-2-220:~$ sudo ufw status
Status: active
To                         Action      From
...                        ......      ....
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)</pre>

3. Then, I changed SSH port to use 2200 in the file `/etc/ssh/sshd_config`.

4. I created a new user called `grader` and provided sudo access to the user by adding a file in `/etc/sudoers.d/grader` and giving the required sudo settings.

4. I also created an ssh key pair for grader using ssh-keygen. Using this, I enforced key based authentication and disabled login through password by again changing the file `/etc/ssh/sshd_config`.

5. I installed postgresql DB and created a user called `catalog` who had no write permissions to other DBs and cound not create any new DBs. I also created a DB called catalog and changed the owner to catalog. Thus, the user catalog was the owner of the DB catalog and could create tables only inside that DB.

6. Next, I installed Apache and added a new virtualhost conf file at `/etc/apache2/sites-enabled/lightsail.conf`.
This had the configs of the two routes for my two applications: `/MusicalInstruments` and `/NeighborhoodMap`. I also added the corresponding projects in `/var/www/` folder by installing git. Since the original project used `sqlite`, I had to change the DB connection to use `postgresql`. I could achieve it using the following:
<pre>create_engine('postgresql://catalog:catalog@localhost:5432/catalog')</pre>
I also installed flask and virtualenv for the catalog app.

7. To make Oauth2 work with providers like Facebook and Google, I had to add the full domain name to the list of allowed domains in my facebook and google developer apps.

### Resources and tutorials used:

1. Digital Ocean 1: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
2. Digital Ocean 2: https://www.digitalocean.com/community/questions/running-mutliple-flask-application
3. Digital Ocean 3: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
4. SQLAlchemy.org: http://docs.sqlalchemy.org/en/latest/core/engines.html
5. Udacity Tutorials.

### Caveats:

Since google and facebook oauth2 apps require full domain name instead of a public IP, please use [`http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com`](http://ec2-52-24-122-174.us-west-2.compute.amazonaws.com) instead of [`52.24.122.174`](52.24.122.174) to test the catalog app.
