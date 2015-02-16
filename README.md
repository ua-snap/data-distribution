# Utility and documentation for SNAP data distribution

## Setting up CKAN for development

 1. Run the Vagrant provisioner located [here](https://github.com/ua-snap/ckan-puppet-centos) to build the base image.
 1. `vagrant ssh` to log into the machine.  For convenience while development, I'll assume all further commands are run as root.
 1. Clone the [ckanext-snap_harvester](https://github.com/ua-snap/ckanext-snap_harvester) repository to `/usr/lib/ckan/default/src`.
 1. Clone the [ckanext-snap_theme](https://github.com/ua-snap/ckanext-snap_theme) repository to `/usr/lib/ckan/default/src`.
 1. Edit the main CKAN configuration file in `/etc/ckan/default/production.ini` and add the `snap_harvester` and `snap_plugins` to the line where additional plugins are configured.

### Running CKAN in a development mode

Instead of serving CKAN through Apache's `mod_wsgi`, it's better to run with the development server because it allows enhanced debug modes to be activated.

 - disable Apache
 - disable supervisord
 - add "debug=True" to config file



