httpd_core
=========

Version: 0.0.4

Role for managing core configuration for Apache HTTPD. Meant to be part of a modular configuration management approach.

Provides a directory structure, a core httpd.conf file, and additional configuration files to handle a core setup. It optionally provides default vhost entries so that the role can result in a minimally functional environment.

Separate roles or playbook task can load additional modules by adding config files to conf.modules.d, extra server-wide configurations by adding config files to conf.d, and vhost entries by adding config files to vhosts. 

TODO:

- If other installations besides the vanilla RHEL rpm is supported, consider recasting the RedHat 6 and RedHat 7 configs as httpd 2.2 and httpd 2.4, respectively.

Requirements
------------

- Assumes httpd is already installed, that the equivalent of /etc/sysconfig/httpd is directing it to the path of the httpd.conf managed by this role, and that init/systemd is set up for it.
- When ssl is in use (default), ssl certificate has been generated and placed in the location specified in httpd_certdir.
- If default vhosts are disabled, something else is setting at least one valid vhost.

Role Variables
--------------

**The following variables can be modified with any value that is compatible with the installed version of Apache HTTPD (2.2 for RHEL 5 & 6 and 2.4 RHEL 7) and related system dependencies**

- httpd_basedir
  - *Default value = /opt*
  - The base directory for your configuration path. The defaults for the following reference this variable:
    - httpd_certdir
    - httpd_serverroot
    - httpd_docroot
  - ** The role does not enforce the existence of this path. If you override this value, make sure your playbook also creates this directory **
- httpd_user
  - *Default value = htadmin*
  - The value provided to the User directive in httpd.conf
  - The Unix filesystem user that owns files and directories created by the role.
- httpd_group
  - *Default value = htadmin*
  - The value provided to the Group directive in httpd.conf
  - The Unix filesystem group that owns files and directories created by the role.
- httpd_dirmode
  - *Default value = 0750*
  - The mode used for all directories created by this role.
  - The only exception is for default docroot, when a value is set for that parameter (see below).
- httpd_mode
  - *Default value = 0640*
  - The mode set for all files created by this role.
- httpd_servicename
  - *Default value = httpd*
  - Name of the Apache HTTPD service, used by init or systemd
- httpd_hostname
  - *Default value = inventory_hostname*
  - The value passed to the ServerName directive in httpd.conf.
  - Simply provides a server-level default. Any ServerName provided in vhost declarations override this for that vhost.
- httpd_serverroot
  - *Default value = {{ httpd_basedir }}/httpd*
  - The path provided to the ServerRoot directive. The base/root directory for the httpd config directories.
- httpd_custom_modules_dir
  - *Default value = httpd_serverroot/custom_modules*
  - A directory where modules not provided through the OS package system can be installed.
  - Provided to give a default solution for where to put module files, so it need not be reinvented for every use case.
- httpd_log_dir
  - *Default value = {{ httpd_serverroot }}/logs/httpd*
  - Directory for holding server logs.
  - The config files use this path directly, but a convenience symlink from httpd_serverroot/logs to this directory is still provided.
- httpd_timeout:
  - *Default value = 60*
  - Timeout setting in httpd.conf
- httpd_keepalive
  - *Default value = 'On'*
  - Keepalive setting in httpd.conf
- httpd_keepalive_maxreqs
  - *Default value = '100'*
  - Max keepalive requests setting in httpd.conf
- httpd_keepalive_timeout
  - *Default value = '5'*
  - Keepalive timeout setting in httpd.conf
- httpd_charset_default
  - *Default value = 'UTF-8'*
  - Default charset value from httpd.conf
- httpd_allow_plaintext
  - *Default value = true*
  - Boolean flag that determines if a non-encrypted listener is opened.
  - Labeled "plaintext" to reflect that encrypted traffic should be taken as the norm.
  - If using default vhost, the default plaintext vhost redirects traffic to ssl.
  - There is another flag, documented below, that instead makes the default plaintext vhost serve the docroot without a redirect.
- httpd_plaintext_port
  - *Default value = 80*
  - Default port for unencrypted http trafic. Can be changed if alternate port is needed.
- httpd_use_ssl
  - *Default value = true*
  - Boolean flag that determines if mod_ssl is loaded and ssl.conf provided for configuration.
- httpd_ssl_port
  - *Default value = 443*
  - Default ssl port for http. Can be changed if alternate port is needed.
- httpd_ssl_protocols
  - *Default value = all -SSLv2 -SSLv3*
  - The allowed set of SSL/TLS protocols. 
  - Only TLS allowed, but allowing TLSv1 and TLSv1.1 since we can't take modern clients as a given.
- httpd_ssl_ciphers
  - *Default value = ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS*
  - Used in ssl.conf. The set of ciphers allowed and excluded, in priority order.
  - Based on https://mozilla.github.io/server-side-tls/ssl-config-generator/ and the range of ciphers currently supported in the Netscaler.
  - Targets "intermediate" profile, due to the presence of older web clients.
- httpd_defaultcert
  - *Default value = {{ httpd_certdir }}/httpd_hostname.crt*
  - Certificate for SSL connections. Only used in the default sll vhost.
- httpd_defaultkey
  - *Default value = {{ httpd_certdir }}/httpd_hostname.key*
  - Private key for SSL connection. Only used in the default ssl vhost.
- httpd_default_docroot_dir
  - *Default value = {{ httpd_basedir }}/docroot*
  - The docroot directory used by the default vhost.
  - This directory is only enforced if default vhosts are enabled.
- httpd_default_docroot_group
  - *Default value = httpd_group*
  - The group ownership for the default docroot directory.
  - Is the same as what is set for httpd_group unless a value is specifically set to this parameter.
- httpd_default_docroot_mode
  - *Default value = httpd_dirmode*
  - The mode for the default docroot directory.
  - Is the same as what is set for httpd_dirmode unless a value is specifically set to this parameter.
- httpd_default_vhost
  - *Default value = true*
  - Boolean flag that determines if default vhost file(s) are provided by this role.
  - Set this to false unless the very minimal vhost declaration provided fits your use case.
  - If set to false, Apache HTTPD will not be able to start until a vhost file has been set by other means.
- httpd_default_vhost_ssl_redirect
  - *Default value = true*
  - Boolean flag that determines if the plaintext vhost provides a redirect to ssl.
  - If true, the plaintext vhost has no directory directive and only provides a rewriterule that redirects to https.
  - If false, the plaintext vhost declares a docroot and a directory directive in the same way done by the ssl default vhost.
- httpd_serveradmin
  - *Default value = *
  - The contact email provided on error pages. Should be a support contact suitable for the end users of the web service. 


  - This manages the general default for the Apache instance. It can be overridden per vhost in vhost files.

**The following variables have default variables based on the standard RHEL rpm installation. They only need to be updated if a different installation method is used**

- httpd_module_dir
  - *Default value = /usr/lib64/httpd/modules*
  - Where the module files are installed by the packaging system.
  - The role creates a symlink to this directory at httpd_serverroot/modules
- httpd_run_dir
  - *Default value = /var/run/httpd*
  - The location of the pid file. This is a root-owned location, and the package-provided init script/unit file may hardcode this path.
  - RHEL 5 puts the pid file directly under /var/run. This is set in the RedHat_5 var file.
  - The role creates a symlink to this directory at httpd_serverroot/run
- httpd_defaultCA
  - *Default value = /etc/pki/tls/certs*
  - Location of the proper CA trust store. This is the path for the system-provided CA trust, which is managed through Puppet and Satellite.
  - RHEL 5 has a different path (set in the RHEL 5 var file). It also has a different method of providing a CA trust, which is less easily automated.

**Do not provide alternative values for the following. Doing so will break the role-generated config files unless you provide symlinks from the standard relative locations to the ones you set by overriding these variables.**

- httpd_conf_dir
  - *Default value = httpd_serverroot/conf*
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

N/A

License
-------

MIT

Author Information
------------------

Initially written by Kevin White, January 2018.
