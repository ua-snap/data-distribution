# Utility and documentation for SNAP data distribution

## Setting up CKAN for development

Once the Puppet provisioning has finished, log onto the server that was configured and create an administrative user:

```bash
sudo su - ckan
. /usr/lib/ckan/default/bin/activate
cd /usr/lib/ckan/default/src/ckan
paster sysadmin add admin -c /etc/ckan/default/production.ini
```

### Running CKAN in a development mode

Instead of serving CKAN through Apache's `mod_wsgi`, it's better to run with the development server because it allows enhanced debug modes to be activated.

 - disable Apache
 - disable supervisord

To launch the development server:

 * `source /usr/lib/ckan/default/bin/activate`
 * `paster serve /etc/ckan/default/production.ini`

### Running the harvester

#### Create an organization

Before you can create a harvester instance, you need to have at least one organization defined.  Log into CKAN's web interface with the admin user you created earlier, then navigate to organizations by entering the URL `https://site.url/organization` and create one named 'snap'.

#### Configure the harvester

As a logged in admin user, go to `localhost:5000/harvest` and configure a new harvest job:

 * URL: https://athena.snap.uaf.edu/geonetwork/srv/en/csw?request=GetCapabilities&service=CSW
 * Source Type: SNAP GeoNetwork Instance

Other options can be left as default.

To launch the harvesters for development, open a new terminal window and `vagrant ssh` into the instance.  Then:

 * `sudo su -`
 * `source /usr/lib/ckan/default/bin/activate`
 * `/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester     gather_consumer --config=/etc/ckan/default/production.ini &`
 * `/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester     fetch_consumer --config=/etc/ckan/default/production.ini`

When you make changes to the harvester plugin, you'll need to restart the fetch stage.  Ensuring that `supervisord` isn't running is important, or you'll end up with an in-memory version of the harvester instead of the one you're working on.

To actually perform harvesting/reharvesting, do this:

 * Go to `http://localhost:5000/harvest`
 * Pick the SNAP Harvester
 * Click "Admin"
 * Click "Clear" then "Reharvest".  This is the best way to ensure that stale/old jobs aren't getting resumed.

## Post-install configuration on production server

After completing the install of the base CKAN and our extensions, there's some additional steps that need to be performed.

### Configuration in the admin menu

Log in as an administrator, go to the admin tools and choose the "Congfig" tab.  Set this section up this way:

 * Site Tag Logo: `snap_data_art.png`

# Configuration of data download directories

Placeholder *TODO* for the moment.  Need to add instructions on adding `h5ai` and how to configure it and edit template as required.

# Restoring from database backup
Restart the Postgresql service to clear all active connections
`sudo /etc/init.d/postgresql-9.3 restart`

Drop ckan_default, datastore_default and postgres databases as postgres user
```
$ /usr/pgsql-9.3/bin/dropdb ckan_default
$ /usr/pgsql-9.3/bin/dropdb datastore_default
$ /usr/pgsql-9.3/bin/dropdb postgres 
```

Recreate the databases to allow for restoring from a text file
```
$ /usr/pgsql-9.3/bin/createdb postgres
$ /usr/pgsql-9.3/bin/createdb ckan_default
$ /usr/pgsql-9.3/bin/createdb datastore_default
```

You must reindex to have things show up properly through Solr on the Web UI
```
$ sudo su - ckan
$ source default/bin/activate
$(default) paster --plugin=ckan search-index rebuild -c /etc/ckan/default/production.ini
```

Now you can start the CKAN instance again
```
$ sudo su - ckan
$ source default/bin/activate
$(default) paster serve /etc/ckan/default/production.ini
```

