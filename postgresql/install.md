# PostgreSQL Installation and setup for Remote Access

## Disclaimer
I recommend following these instructions at your own risk. They are built with my configuration roughly in mind with certain details scrubbed as to not expose sensitive information.

*This is assuming that you are using Fedora 29 or similar.*

A lot of this setup process was adopted and modified from [this](http://suite.opengeo.org/docs/latest/dataadmin/pgGettingStarted/firstconnect.html) documentation over at [opengeo.org](http://www.opengeo.org).

## Install Packages
```bash
dnf install -y https://download.postgresql.org/pub/repos/yum/11/fedora/fedora-29-x86_64/pgdg-fedora11-11-2.noarch.rpm
dnf update -y
dnf install -y postgresql11-server
```

## Initialize the database
```bash
/usr/pgsql-11/bin/postgresql-11-setup initdb
systemctl enable postgresql-11
systemctl start postgresql-11
```

## Set Postgres User Password
```bash
# Run psql as postgres user
sudo -u postgres psql postgres
# Change password for postgres
\password postgres
# Enter your password twice when prompted
# Exit psql
\q
```

## Allow Connections

### Local Connections
```bash
sudo vi /var/lib/pgsql/11/data/pg_hba.conf
```

Scroll down to the line that describes local socket connections. It may look like this:
```
local   all     all             peer
```

Change `peer` to `md5`.

### Local pgAdmin

Find the line that describes local loopback connections over IPv6:
```
host    all     all     ::1/128     ident
```

Change `ident` to `md5`.

### Remote pgAdmin
```bash
sudo vi /var/lib/pgsql/11/data/pg_hba.conf
```

Find the line that describes local connections. Should look like:
```
local   all     all              md5
```

Below that line, we are going to enter the following to allow authenticated remote connections.

Replace the subnet here with your applicable subnet. You can add multiple smaller subnets if you wish.
```conf
# Remote IPv4 connections:
host	all		all     192.168.0.0/16		md5
```

**NOTE: I'm sure that this is not secure. Make sure that you review the official documentation. This is for development use only!**

Next we are going to configure the server to listen for remote connections on all IP's

```bash
sudo vi /var/lib/pgsql/11/data/postgresql.conf
```

Locate the line 
```conf
#listen_addresses = 'localhost'
```

Change that line to read
```conf
listen_addresses = '*'
```
And save the file.

#### Firewall Setup
Next we are going to enable remote connections through the firewall.

```bash
sudo firewall-cmd --permanent --add-service=postgresql
sudo firewall-cmd --reload
```

## Testing
Now, just test you connection and you should be good to go!