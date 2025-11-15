# Repo Release Download Manager
Easily download, verify, extract, and stage releases from remote repositories.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_repo/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_repo/tree/main/defaults/main/)

### Generated Variables
After successful execution the following variables are available for further
manipulation of extracted release during the same play (standard role variable
scope):

 Variable              | type | Description
-----------------------|------|-----------------------------------------
 _repo_asset           | dict | Asset metadata.
 _repo_changed         | bool | True if new asset extracted/installed.
 _repo_checksum        | dict | Checksum metadata.
 _repo_dir             | str  | Versioned repository extract location.
 _repo_remote_metadata | dict | Release metadata for requested version.
 _repo_signature       | dict | Signature metadata.
 _repo_tag             | str  | Detected tag version.

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.deb](https://github.com/r-pufky/ansible_collection_deb) collection.

## Example Playbook
This role is intended to be called multiple times in other roles with most
options being set per repository release being downloaded. Files may optionally
be migrated to new installations. Multiple remote repositories are supported.

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download and extract latest release (from github)'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.repo'
  vars:
    repo_host_access_token: '{ACCESS TOKEN FROM GITHUB}'
    repo_release_owner: 'sct'
    repo_release_repo: 'overseerr'
    repo_release_tag: 'latest'
    repo_file_owner: 'overseerr'
    repo_file_group: 'overseerr'
    repo_extract_dest: '/opt/overseerr'
    repo_extract_mode: 'a-st,o-rwx'
    repo_extract_extra_opts:
      - '--strip-components=1'
    repo_extract_symlink: '/opt/overseerr/latest'
    repo_extract_migrate_files: ['config.db']
    repo_extract_remove_files:
      - 'README.md'
      - 'config/.keep'
```

Download a binary from codeberg and validate checksums, GPG signatures.

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download binary release (from codeberg)'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.repo'
  vars:
    repo_host: 'codeberg'
    repo_host_access_token: '{ACCESS TOKEN FROM CODEBERG}'
    repo_release_owner: 'forgejo'
    repo_release_repo: 'forgejo'
    repo_release_specifier: 'v'
    repo_release_asset: 'forgejo-{BARE_VERSION}-linux-amd64'
    repo_release_checksum: 'forgejo-{BARE_VERSION}-linux-amd64.sha256'
    repo_release_signature: 'forgejo-{BARE_VERSION}-linux-amd64.asc'
    repo_release_gpg_key_id: 'EB114F5E6C0DC2BCDD183550A4B61A2DC5923710'
    repo_extract_mode: 'a-st,o=rwx,g=rx,o-rwx'
    repo_extract_dir: '/opt/forgejo'
```

## Development
Configure [environment](https://r-pufky.github.io/ansible_collection_docs/ansible/environment)

Run all unit tests:
``` bash
molecule test --all
```

Run integration tests against live API:
``` bash
molecule test -s live_api_test -- -v \
  -e '{"repo_live_api_github_enable":true, "repo_host_access_token": "{TOKEN}"}'
```

### Releases
Major release versions track Debian release versions:

* **[13.x.x](https://github.com/r-pufky/ansible_repo)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_repo/tree/12.x)**: 12 Bookworm.

### Testing in other Roles
Testing may be toggled on the role to facilitate testing other roles which
consume this role, removing the need to hammer REST API's or download archive
files. Never rely on this for live code. Effectively this is a mock.

group_vars/archives:
```
project_v1.0.0.tar.gz
project_v1.0.1.tar.gz
```

roles/my_custom_role/molecule/molecule.yml
``` yaml
provisioner:
  inventory:
    group_vars:
      all:
        repo_testing_enable: true
        repo_testing_versions:
          - comment: '1.0.0 called with checksum and signature validation'
            version: 'v1.0.0'
            asset: 'group_vars/assets/test_archive_v1.0.0.tar.gz'
            checksum: 'group_vars/assets/test_archive_v1.0.0.tar.gz.sha256'
            signature: 'group_vars/assets/test_archive_v1.0.0.tar.gz.asc'
          - version: 'v1.0.1 called with no validation'
            asset: 'group_vars/assets/test_archive_v1.0.1.tar.gz'
```
When the role is called during testing, the first execution of the role will
return `v1.0.0` and the repo download will be simulated by copying the
specified assets from the ansible controller to the testing node.

The Second execution of the role will return `v1.0.1` and the repo download
will be simulated by copying the specified asset from the ansible controller to
the testing node.

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_repo/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
