# acme&middot;sh

Installs [acme.sh](https://api.github.com/repos/acmesh-official/acme.sh) from the latest Release on GitHub.

This Role creates a normal user (called `acmesh` by default), if it does not exist and installes acme.sh as this user, by default in it's home directory. A certificate renewal cronjob is created for this user and a bash alias `acmesh` is defined.

Certificates store is separated from user's home. The certificates store path is configured with `acmesh_cert_dir` (defaults to */etc/acme-certs*). The `acmesh_cert_dir_rights` defines the rights for the certificates directory: group ownership and mode. To set the correct mode for the certificates there is the script *certs-fix-permissions.sh*.

Only the web root mode is supported for automatically certificate issuing.

Calling the role with `acmesh_certs` variable set enables automated certificate management.


## Requirements

* Systempackage `unzip` for Ansibles [`unarchive` module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/unarchive_module.html).
* Systempackage `curl` for acme.sh to work, alternatively `wget`.

_For automatic ensuring of the packages, see variable `acmesh_ensure_requirements`._


## Role Variables

* **`acmesh_state`**  
  Defines which state is to apply. While `present` ensures that acme.sh is installed, `updated` always runs the installation and update routine to determine whether a new version is available or the desired version is installed.  
  Choices: `present`, `updated`  
  Default: `present`

* **`acmesh_version`**  
  Specifies which version to install. While `latest` installs the version of the latest release, specifying a tag name can install a special version and also allows downgrading. `master` installs the version of the master branch.  
  *Note that `acmesh_version: master` in combination with `acmesh_state: updated` is not idempotent.*  
  Choices: `latest`, `master`, `<tag>`  
  Default: `latest`

* **`acmesh_configure`**  
  Allows to limit the frequency of configuration. While `always` runs the configuration every time the role is executed, `on_install` allows the configuration to stop when the role is executed but the application is already installed. The `on_install` setting is useful when the role is executed to issue certificates or disable renewal.  
  Choices: `always`, `on_install`  
  Default: `on_install`

* **`acmesh_ensure_requirements`**  
  Provides for the installation of the packages listed under requirements.  
  Type: bool  
  Default: `no`

* **`acmesh_user`**  
  The user for executing acme.sh.  
  Default: `acmesh`

* **`acmesh_user_home`**  
  The user home directory while creating the user. Home directory is not moved later.  
  Default: *undefined*

* **`acmesh_install_root`**  
  The directory where acme.sh should be installed.  
  Default: `~/acmesh`

* **`acmesh_scripts_dir`**  
  The directory for scripts, shipped with *acmesh* role.  
  If it is a relative path, `acmesh_install_root` is assumed to be the base path.  
  Default: `scripts`

* **`acmesh_cert_dir`**  
  The directory for the certificates.  
  Default: `/etc/acme-certs`

* **`acmesh_challange_dir`**  
  The directory for the acme-challange on `.well-known/acme-challange`.  
  Default: `/var/www/acme-challange`

* **`acmesh_cert_dir_rights`**  
  Controles the ownership and mode for the certificates directory.  
  Sub keys are: `group`, `mode`.  
  Type: Dict  
  Default:
  ```yml
  acmesh_cert_dir_rights:
    group: "{{ acmesh_user }}"
    mode: '0750'
  ```
  When `group` is defined, it makes sense to set `mode` to `02750`.  
  Example:
  ```yml
  acmesh_cert_dir_rights:
    group: www-data
    mode: '02750'
  ```

* **`acmesh_certs_mode`**  
  Mode for elements in the certificates directory. Values are templated to *certs-fix-permissions.sh* script, used as value for *chmod* command.  
  Sub keys are: `dirs`, `files`.  
  Type: Dict  
  Default: `750` (dirs), `640` (files)

### Configuration

* **`acmesh_default_ca`**  
  Configures the default CA when requesting certificates without the explicit specification of the server. Could be a [Short Name](https://github.com/acmesh-official/acme.sh/wiki/Server), defined by acme.sh or a URL.  
  Default: `letsencrypt`

* **`acmesh_default_chain`**  
  Configures the default Chain for a specific CA (server).  
  Type: List of Dicts  
  Default:
  ```yml
  - server: letsencrypt
    chain: ISRG
  ```

* **`acmesh_cron_state`**  
  Whether the cronjob should be present or absent.  
  Choices: `present`, `absemt`  
  Default: `present`

* **`acmesh_cron_hour`**  
  The hour value for the cronjob. Defaults to `*` if it is unset or empty.  
  Default: `VAL`

* **`acmesh_cron_minute`**  
  The minute value for the cronjob. Defaults to `0` if it is unset or empty.  
  Default: `VAL`

* **`acmesh_cron_weekday`**  
  The weekday value for the cronjob. Defaults to `*` if it is unset or empty.  
  Default: `VAL`

* **`acmesh_cron_day`**  
  The day value for the cronjob. Defaults to `*` if it is unset or empty.  
  Default: *undefined*

* **`acmesh_bash_aliases`**  
  Whether the bash aliases should be present or absent.  
  Choices: `present`, `absemt`  
  Default: `present`

### Certificate handling

* **`acmesh_keylength_rsa`**  
  Used keylenght if `type` is defined as `rsa` or `both` (when rsa cert is issued). Concerns certificate issue with `acmesh_certs`.  
  Choices: *see acme.sh documentation*  
  Default: `4096`

* **`acmesh_keylength_ecc`**  
  Used keylenght if `type` is defined as `ecc` or `both` (when ecc cert is issued). Concerns certificate issue with `acmesh_certs`.  
  Choices: *see acme.sh documentation*  
  Default: `ec-384`

* **`acmesh_default_cert_test`**  
  Defines if issued certificates are issued as test certificates or not. This is the fallback if `test` in the `acmesh_certs` list is not set, so `test` setting in `acmesh_certs` list has a stronger effect.  
  Type: bool  
  Default: `no`

* **`acmesh_ignore_cert_errors`**  
  Specifies whether errors in the certification command are ignored. If errors are not ignored, the entire playbook execution stops in case of an error.  
  Type: bool  
  Default: `yes`

* **`acmesh_certs`**  
  The certificates that are to be issued or whose renewal is to be disabled.  
  Type: List of Dicts  
  Default: `[]`  
  Sub items (Dict):
  * `domains`  
    List of domains. First domain is primary.
  * `state`  
    Desired state of the certificate.  
    Choices: `issued`, `renewed`, `disabled`  
    Default: `issued`
  * `type`  
    Type of the certificate. For keylength defaults see `acmesh_keylength_rsa` and `acmesh_keylength_ecc`.  
    Choices: `rsa`, `ecc`, `both`  
    Default: `rsa`
  * `force`  
    Forces the certificate renewing if it's enabled. Parameter `--force` from acme.sh.  
    Type: bool  
    Default: `false`
  * `test`  
    Uses the test/staging area from the CA if it's enabled and supported by the CA. Parameter `--test` from acme.sh.  
    Type: bool  
    Default: `false`
  * `server`  
    CA server for issuing.
  * `keylength`  
    Allows to set specific keylength, overwrites `type`, even if `type: both`.  
    Choices: *see acme.sh documentation*

<!--
* **`VAR`**  
  DESC  
  Choices: `VAL`, `ANOTHER`  
  Default: `VAL`

* **`VAR`**  
  DESC  
  Type: List of Dicts  
  Default: `VAL`
-->

## Dependencies

None.


## Example Playbook

Make sure that acme.sh is installed and configured to use Let's Encrypt.

```yml
---
- hosts: webservers
  roles:
    - xolyu.acmesh
```

Update acme.sh to the latest release.

```yml
---
- hosts: webservers
  roles:
    - role: xolyu.acmesh
      acmesh_state: updated
```

Configure acme.sh to another default CA after the installation has been completed.

```yml
---
- hosts: webservers
  roles:
    - role: xolyu.acmesh
      acmesh_configure: always
      acmesh_default_ca: zerossl
```

Issue an RSA test certificates for the server.

```yml
# playbook.yml
---
- hosts: webserver

  vars:
    acmesh_default_cert_test: yes
    acmesh_certs:
      - domain:
          - "{{ ansible_fqdn }}"
      - domain:
          - myblog.example.com

  roles:
    - xolyu.acmesh
```

Issue an ECC certificate with a task by using `include_role`.

```yml
- name: Issue certificate.
  include_role:
    name: xolyu.acmesh
  vars:
    acmesh_certs:
      - domains:
          - dummy.example.com
          - www.dummy.example.com
        type: ecc
```


## License

GNU Lesser General Public License v3.0 only


## Author Information

Xolyu.
