# Utility and documentation for SNAP data distribution

## Setting up CKAN for development

 1. Run the Vagrant provisioner located [here](https://github.com/ua-snap/ckan-puppet-centos) to build the base image.
 1. `vagrant ssh` to log into the machine.  For convenience while development, I'll assume all further commands are run as root.
 1. `source /usr/lib/ckan/default/bin/activate`
 1. Clone the [ckanext-snap_harvester](https://github.com/ua-snap/ckanext-snap_harvester) repository to `/usr/lib/ckan/default/src`.
 1. `cd /usr/lib/ckan/default/src/ckanext-snap_harvester`
 1. `python setup.py build`
 1. `python setup.py install`
 1. Clone the [ckanext-snap_theme](https://github.com/ua-snap/ckanext-snap_theme) repository to `/usr/lib/ckan/default/src`.
 1. `cd /usr/lib/ckan/default/src/ckanext-snap_theme`
 1. `python setup.py build`
 1. `python setup.py install`
 1. Edit the main CKAN configuration file in `/etc/ckan/default/production.ini` and add the `snap_harvester` and `snap_theme` to the line where additional plugins are configured.

### Running CKAN in a development mode

Instead of serving CKAN through Apache's `mod_wsgi`, it's better to run with the development server because it allows enhanced debug modes to be activated.

 - disable Apache
 - disable supervisord
 - add "debug=True" to config file

In order to use the additional debug interface tools, you need to copy a CSS file or CKAN will choke.

 * `cp /usr/lib/ckan/default/src/ckan/ckan/public/base/css/main.css`
 * `/usr/lib/ckan/default/src/ckan/ckan/public/base/css/main.debug.css`

To launch the development server:

 * `source /usr/lib/ckan/default/bin/activate`
 * `paster serve /etc/ckan/default/production.ini`

### Running the harvester

Before you can create a harvester instance, you need to have at least one organization defined.  Go to the Organizations menu at the top of the screen and create one named 'snap'.

Go to `localhost:5000/harvest` and configure a new harvest job:

 * URL: https://athena.snap.uaf.edu/geonetwork/srv/en/csw?request=GetCapabilities&service=CSW
 * Source Type: SNAP GeoNetwork Instance

Other options can be left as default.

Next, modify the default cronjob to run once per minute so you don't have to wait too long for the harvesting to kick off.

 * `crontab -u ckan -e`
 * Change the last line to: `/1 * * * * /usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester run --config=/etc/ckan/default/production.ini`

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


