# froxlor-installer
This is a new version of the froxlor installer script. The main goal of this new script is to allow re-configuration at any time instead of one-shot solutions hence it is designed to be idempotent for most tasks.

# Usage
Bash install script to install latest version of froxlor including all required system components.
Optionally skip the installation part and just use it to configure an existing system.

| Distribution | Version | Supported |
|--------------|---------|-----------|
| Debian       | 11      | &cross;   |
| Debian       | 12      | &check;   |
| RHEL         | 8       | &cross;   |
| RHEL         | 9       | &check;   |
| FreeBSD      | 12      | &cross;   |
| FreeBSD      | 13      | &check;   |

## Installation
Make sure to have `git` installed.  
On FreeBSD you can speedup the process by running `portsnap fetch` in upfront. Additionally, it might required to [switch to the latest package branch](https://docs.freebsd.org/en/books/handbook/ports/#quarterly-latest-branch).  
When configuring your system during the initial install process, *do not enable SSL in froxlor web* because otherwise the system service configuration will fail as no SSL certificate has been generated by Let's Encrypt yet.  
Login to your machine, open terminal (**ssh/shell**) and run the following command **as root**:   
```shell
mkdir -p /etc/froxlor/installer && \
git clone --quiet https://github.com/thijsvanderwoude/froxlor-installer.git /etc/froxlor/installer && \
sh /etc/froxlor/installer/install.sh
```
Remember to exchange `/etc/` by `/usr/local/etc/` when using FreeBSD.

## Configuration
Most configuration details are obtained directly from the froxlor database hence you can make most configurations from the froxlor webpanel. Some services cannot be configured in  froxlor webpanel, however. You can find those config files in `/etc/froxlor/`. Edit services settings here and not directly at the service config file, else your modifications might be overwritten on next run.

## Reconfigure
Everytime you have changed service configuration in froxlor web and you want to apply them to your system run:  
```shell
sh /etc/froxlor/installer/install.sh
```

## Updating
If you want to update the configuration templates pull the newest version here from github and re-run the configuration utility:
```shell
git -C /etc/froxlor/installer pull
sh /etc/froxlor/installer/install.sh
```

## Configuration only setup
To use the configuration functionality only without a new full installation, some manual pre-configurations are required which were set during a full install automatically.
```shell
cat > ~/.froxlor.cnf <<EOF
[client]
# The following password is used by all standard MySQL clients
password=<your password for unprivileged froxlor user>
user=<username of unprivileged froxlor user>
host=localhost
database=<database name of froxlor database>
EOF
```

```shell
cat > ~/.froxroot.cnf <<EOF
[client]
# The following password is used by all standard MySQL clients
password=<your password for privileged froxlor user>
user=<username of privileged froxlor user>
host=localhost
database=<database name of froxlor database>
EOF
```

```shell
touch /etc/froxlor/installer/INSTALLED
EOF
```

## ~~Configure specific app~~ NOT WORKING
~~Running all tasks everytime you have changed a configuration in froxlor does not break anything but you can save time by executing the configuration utility for the app you changed settings only.~~  
~~Run the following command **as root**:~~  
~~`cd /etc/froxlor && ansible localhost -m include_role -a name=< ftp | webserver | php >`~~  
THIS IS NOT WORKING RIGHT NOW. USE FULL RECONFIGURATION INSTEAD.

# Disable additional features
This script configures certain features which are not included in the standard froxlor configuration. It usually doesn't break anything when having them enabled but if you experience errors or malefunctionalities with your web apps for instance, these features can be disabled. The ost practicable is to modify `/etc/froxlor/froxlor.conf` after a successfull installation and run the configuration script again.  
If you have assigned a custom ssh port (which is not recommended: no security by obscurity), you should disable the firewall feature before configuring the system:
```shell
cat > /etc/froxlor/froxlor.conf <<EOF
firewall:
  enable: false
EOF
```
It is recommended to make modifications to the config files in `/etc/froxlor` after the initial installation has finished.
Remember to exchange `/etc` by `/usr/local/etc` when using FreeBSD.

# Limitations
PowerDNS with Bind9 backend isn't working.  
phpMyAdmin can only be installed into the webroot (configure phpMyAdmin-URL setting in webpanel to `https://<domain>/phpmyadmin`).
If the webserver enablement is changed, the configuration rebuild needs to be triggered manually in froxlor web too.  
Only PHP-FPM is supported currently. Configure your installation accordingly in the webpanel.
Only Dovecot is supported currently, not courier.

# Next Milestones
Creating an Ansible module for froxlor which allows direct interaction with Froxlor API  
Better SSL integration for Email  
Firewalld zone profiles  
ModSecurity3
