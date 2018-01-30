httpd_core
=========

A brief description of the role goes here.

Requirements
------------

- Assumes httpd is already installed, that the equivalent of /etc/sysconfig/httpd is directing it to the path of the httpd.conf managed by this role, and that init/systemd is set up for it.
- When ssl is in use (default), ssl certificate has been generated and placed in the location specified in httpd_certdir.


Role Variables
--------------

**Do not provide alternative values for the following. Doing so will break the role-generated config files unless you provide symlinks from the standard relative locations to the ones you set by overriding these variables.**

- httpd_conf_dir
  - _Default value = httpd_serverroot/conf_
  - the location where httpd.conf and the 'magic' file go.
- httpd_confd_dir
  - *Default value = httpd_serverroot/conf.d*
  - the location for configurations (in .conf files) that do not belong in the core httpd.conf file.
- httpd_vhost_dir
  - *Default value = httpd_serverroot/vhosts*
  - the location for .conf files that define vhosts
- httpd_moduleconf_dir
  - *Default value = httpd_serverroot/conf.modules.d*
  - the location for .conf files that determine which httpd modules to load.

**The following can be changed, but the only effect would be to use the defaults for a different os version**
- httpd_osver
  - Value is a pair of ansible facts (ansible_os_family and ansible_distribution_major_version) connected by an underscore.
  - Currently supported values are RedHat_6 and RedHat_7
  - Used to indicate which config file/template and vars file are used by the role.

**Defined in role vars by os version, can only be overridden by being directly passed as paramters with the role at invocation.**
**Should not generally need to be changed.**
 - httpd_is_systemd
   - *Default value = true for RedHat_7 and false for RedHat_6*
   - Is systemd the init system used on the server? Affects when commands go in the restart scripts.

Dependencies
------------

N/A

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

MIT

Author Information
------------------

Initially written by Kevin White, January 2018.

httpd_user *Default value = *
httpd_group *Default value = *


httpd_dirmode *Default value = 0750*
httpd_mode *Default value = 0640*

httpd_servicename *Default value = httpd*
httpd_hostname *Default value = inventory_hostname*

httpd_serverroot *Default value = /opt/httpd*
httpd_custom_modules_dir *Default value = httpd_serverroot/custom_modules*
httpd_log_dir *Default value = /opt/httpd/logs/httpd*

httpd_module_dir *Default value = /usr/lib64/httpd/modules*
httpd_run_dir *Default value = /var/run/httpd*

httpd_default_docroot_dir *Default value = /opt/docroot*
httpd_default_docroot_group *Default value = *
httpd_default_docroot_mode *Default value = httpd_dirmode*

httpd_default_vhost *Default value = true*

if true, default plaintext vhost will simply redirect to ssl
requires httpd_allow_plaintext=true,
or else there won't be a non-ssl default vhost
httpd_default_vhost_ssl_redirect *Default value = true*

httpd_allow_htaccess *Default value = false*

allow a plaintext (non-ssl) listener to be opened
httpd_allow_plaintext *Default value = true*
httpd_plaintext_port *Default value = 80*

enable use of ssl
httpd_use_ssl *Default value = true*
httpd_ssl_port *Default value = 443*

httpd_ssl_protocols *Default value = all -SSLv2 -SSLv3*
httpd_ssl_ciphers *Default value = ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS*

httpd_defaultcert *Default value = /opt/ssl/httpd_hostname.crt*
httpd_defaultkey *Default value = /opt/ssl/httpd_hostname.key*
httpd_defaultCA *Default value = /etc/pki/tls/certs*

httpd_serveradmin *Default value = *
