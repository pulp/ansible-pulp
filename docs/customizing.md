Customizing Your Pulp Installation
==================================

Background info on Ansible variables
------------------------------------

Because the pulp installer is based on Ansible it inherits 2 Ansible designs:

1. Variables can be set in many different ways by the user.
2. Each role has its own set of variables.

This section (and the docs in general) will not cover every possible way of [setting variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#where-to-set-variables), but instead cover the
simplest way: in the playbook itself.

This section will not cover every single variable, because there so many. They are completedly
documented in every single role's README.

Note that variables are yaml and have types such as enums. JSON can also be used as values to the variables.

Pulp Installer's types of Variables
-----------------------------------
Essentially, there are 3 categories of variables
1. Those that make pulp_installer compatible with your environment. For example, the firewall.
2. Preferences for how Pulp is configured/deployed. For example, which plugins to install.
3. Optional variables to make pulp_installer reproducible as newer versions of plugins are released.

WRT to #2, the large data structure variables are `pulp_settings` & `pulp_install_plugins`.
WRT to #3, the large data structure variable is `pulp_install_plugins`.

Because the Pulp installer is composed of Ansible roles, you can use the variables for each of these roles to customize your Pulp installation.
For example, if you want to specify firewall requirements, edit the corresponding firewall variables associated with the [pulp_webserver](https://pulp-installer.readthedocs.io/en/latest/roles/pulp_webserver/#pulp_webserver) role.
Each role documents all the variables that it uses. Some variables are
used by multiple roles. In that case, they are documented in their primary role and mentioned in
the `shared_variables` section the other roles.

**Required Variables:**
Most variables have sane defaults but a few are required. See ``playbooks/example-use/group_vars/all`` for
the minimal set of required variables.

Example of setting a multitude of variables:
--------------------------------------------
In the following example, a multitude of variables are set:
`pulp_settings` is set to talk to object storage.

`variable_name`, a variable documented in `role_name`, is set to `foo`.

`pulp_install_plugins` is set to the plugins we wish to install.

Note on Plugin Version Compatibility with Pulpcore
--------------------------------------------------

Pulp 3 has a plugin architecture so that new content types and features can be added by the
larger community. However, both pulpcore & plugins are installed via pip, which has limited
dependency resolution. Plugins release at their own lifecycles. Thus in the worst case scenario, the
2 stable branches of plugin pulp_juicy could depend on the current branch of pulpcore, while the 2
stable branches of pulp_sugary could depend on an older branch of pulpcore.

In order to avoid breaking multiple plugins for the sake of one plugin, and to avoid breaking existing
installations, upgrading a plugin will not cause pulpcore to be updated as a dependency. Similarly,
if there is an attempt to update a plugin to a version that is incompatible with pulpcore, the installer
will fail and exit. The installer does a compatibility check early in the installation to prevent Pulp
from being installed or upgraded to an incompatible state.

Thus you, yourself, must research plugin compatibility with the pulpcore version whether you are
installing one or more plugins.

Recommended Workflows for Pulpcore & Plugin Versioning
------------------------------------------------------

### Latest Version with Minimal Effort:

Initial installation:

1. Make sure you are running the latest version of the installer, which installs the latest version
   of pulpcore (3.12.1).
1. Confirm that all the latest stable releases of your desired plugins are compatible with pulpcore
   3.12.1, such as by reading the release announcement email thread for pulpcore 3.12.1, reading the
plugins README, or as a last resort, reading their `setup.py`.
1. Set `pulp_install_plugins` with each plugin listed as a key, with an empty enum '{}' as each plugin's value.
1. Run `pulp_installer`.
1. Make sure to save your variables/playbook for later usage.

Example `pulp_install_plugins`:
```
  vars:
    pulp_install_plugins:
      pulp-container: {}
      pulp-file: {}
      pulp-rpm: {}
```

Upgrading your installation:

1. Observe what is the latest version of `pulp_installer`, and what version of pulpcore it installed
   (3.12.1).
1. Confirm that all the latest stable releases of **currently installed** plugins are compatible
   with pulpcore 3.12.1, such as by reading the release announcement email thread for pulpcore 3.12.1,
reading the plugins README, or as a last resort, reading their setup.py.
1. If they are not all compatible yet, **wait** for the plugins to be updated for
   compatibility.
1. Upgrade `pulp_installer` to the latest version.
1. Set `pulp_install_plugins` with each plugin listed as a key, and with each plugin having a key under it called `upgrade` set to the value `true`.
1. Re-run `pulp_installer`.
1. Make sure to save your variables/playbook for later usage.

Example `pulp_install_plugins`:
```
  vars:
    pulp_install_plugins:
      pulp-container:
         upgrade: true
      pulp-file:
         upgrade: true
      pulp-rpm:
         upgrade: true
```

### Specifying Exact Versions with Reproducibility:

Initial installation:

1. Observe the latest branch of `pulp_installer`, and what version of pulpcore it installs (3.12.1).
1. Confirm that all the latest stable releases of your desired plugins are compatible with pulpcore
   3.12.1, such as by reading the release announcement email thread for pulpcore 3.12.1, reading the
plugins README, or as a last resort, reading their setup.py.
1. If they are not all compatible yet, try the last version of the installer that installs pulpcore
   3.11.z . Then confirm that there exist stable releases of your desired plugins that are compatible
with pulpcore 3.11.z. If there are none, try pulpcore 3.10.z, and repeat.
1. Once a compatible pulpcore version is found, Set `pulp_install_plugins` with each plugin listed as a key, and with each plugin having a key under it called `version` set to the version as the value (with quotes).
1. Run `pulp_installer`
1. Make sure to save your variables/playbook for later usage.

Example `pulp_install_plugins` (with bogus version values):
```
  vars:
    pulp_install_plugins:
      pulp-container:
         version: "4.5.6"
      pulp-file:
         version: "5.6.7"
      pulp-rpm:
         version: "6.7.8"
```


Upgrading your install:

1. Observe what the latest version of `pulp_installer` is, and what version of pulpcore it installed
   (3.12.1). (Even if there is no update, you can still upgrade your plugins.)
1. Confirm that all the latest stable releases of **currently installed** plugins are compatible
   with pulpcore 3.12.1, such as by reading the release announcement email thread for pulpcore 3.12.1,
reading the plugins README, or as a last resort, reading their setup.py.
1. If they are not all compatible yet, try the last version of the installer that installs pulpcore
   3.11.z . Then confirm that there exist stable releases of your desired plugins that are compatible
with pulpcore 3.11.z. If there are none, try pulpcore 3.10.z, and repeat.
1. Once a compatible pulpcore version is found, Set `pulp_install_plugins` with each plugin listed as a key, and with each plugin having a key under it called `version` set to the **new** version as the value (with quotes).
1. Upgrade `pulp_installer` to the latest version (if there is a new version.)
1. Run `pulp_installer`
1. Make sure to save your variables/playbook for later usage.


Example `pulp_install_plugins` (with bogus version values):
```
  vars:
    pulp_install_plugins:
      pulp-container:
         version: "5.6.7"
      pulp-file:
         version: "6.7.8"
      pulp-rpm:
         version: "7.8.9"
```