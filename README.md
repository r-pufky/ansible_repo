# Github Release Download Manager
Easily download and extract latest source releases from github repositories.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_github/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_github/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_github/tree/main/defaults/main/)

### Generated Variables
After successful execution the following variables are available for further
manipulation of extracted release during the same play (standard role variable
scope):

 Variable                | type | Description
-------------------------|------|-----------------------------------------
 _github_target          | str  | Full version release tag.
 _github_target_url      | str  | Download url for target release.
 _github_archive         | str  | Local versioned archive location.
 _github_dir             | str  | Local versioned extract location.
 _github_remote_metadata | dict | Release metadata for requested version.
 _github_changed         | bool | True if new archive extracted/installed.

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
This role is intended to be called multiple times in other roles with most
options being set per repository release being downloaded. Files may optionally
be migrated to new installations.

group_vars/all/main.yml
``` yaml
# Accessible to all hosts which will consume this role.
github_personal_access_token: '{ACCESS TOKEN FROM GITHUB}'
```

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download and extract latest release'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.github'
  vars:
    github_repo_release_owner: 'sct'
    github_repo_release_repo: 'overseerr'
    github_repo_release: 'latest'
    github_repo_archive_type: 'tar.gz'
    github_file_owner: 'overseerr'
    github_file_group: 'overseerr'
    github_extract_dest: '/opt/overseerr'
    github_extract_mode: 'a-st,o-rwx'
    github_extract_extra_opts: '--strip-components=1'
    github_extract_symlink_target: 'opt/overserr/latest'
    github_extract_migrate_files: ['config.db']
    github_extract_remove_files:
      - 'README.md'
      - 'config/.keep'
```

Binary releases may be explicitly specified for repositories which release
source and binaries:

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download and extract latest BINARY release'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.github'
  vars:
    github_binary_enable: true
    github_binary_specifier: 'v'
    github_binary_url: '{VERSION}/Binary.{BARE_VERSION}.x86-64.tar.gz'
    github_repo_release_owner: 'sct'
    github_repo_release_repo: 'overseerr'
    github_repo_release: 'latest'
    github_repo_archive_type: 'tar.gz'
    github_file_owner: 'overseerr'
    github_file_group: 'overseerr'
    github_extract_dest: '/opt/overseerr'
    github_extract_mode: 'a-st,o-rwx'
    github_extract_extra_opts: '--strip-components=1'
    github_extract_symlink_target: 'opt/overserr/latest'
    github_extract_migrate_files:
      - 'config.db'
      - 'data_directory'
    github_extract_remove_files:
      - 'README.md'
      - 'config/.keep'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

Run integration tests against live API:
```
molecule test -s live_api_test -- -v -e 'github_testing_enable=true,github_personal_access_token={TOKEN}'
```

### Testing in other Roles
Testing may be toggled on the role to facilitate testing other roles which
consume this role, removing the need to hammer the github REST API or download
archive files. Never rely on this for live code. Effectively this is a mock.

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
        github_testing_enable: true
        github_testing_versions:
          - 'v1.0.0'
          - 'v1.0.1'
        github_testing_archives:
          - 'group_vars/archives/project_v1.0.0.tar.gz'
          - 'group_vars/archives/project_v1.0.1.tar.gz'
```
When the role is called during testing, the first execution of the role will
return `v1.0.0` and the github download will be simulated by synchronizing the
specified archive from the ansible controller to the testing node.

The Second execution of the role will return `v1.0.1` and the github download
will be simulated by synchronizing the specified archive from the ansible
controller to the testing node.

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_github/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
