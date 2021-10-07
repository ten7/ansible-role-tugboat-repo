# Ansible Role: tugboat_repo

This role allows you to programmatically create and manage repositories on Tugboat.qa.

## Requirements

* You must have a Tugboat API key.
* The Tugboat API must be accessible.

## Role Variables

Available variables are listed below.

### API Token

```
tugboat_api_token: '1234567890abscdefg'
```

Your Tugboat API token.

### Creating repositories

This role only works with on repository at a time. You may, however, run it multiple times in the same playbook with an `include_role`.

```yaml
tugboat_repo:
  project: "my-clients"
  name: 'example.com'
  state: present
```

Where:
* **project** is the project under which to create the repository. Required.
* **name** is the repository name to manage. Required.
* **state** is if the domain is `present` or `absent`. Optional, defaults to `present`.

### Specifying repository details

You may specify other repository details as follows

```yaml
tugboat_repo:
  project: "my-clients"
  name: 'example.com'
  state: present
  autobuild: true
  autodelete: true
  autorebuild: true
  build_timeout: "1200"
  envvars:
    - name: ANSIBLE_VERBOSITY
      value: "4"
  domain: "example.com"
  ip_allow:
    - "10.2.3.4"
    - "192.168.1.0/24"
    - "fe80::/64"
  ip_deny:
    - "10.2.3.4"
    - "192.168.1.0/24"
    - "fe80::/64"
  provider_deployment: false
  provider_forks: false
  quota: "0"
  rebuild_orphaned: false
  rebuild_stale: false
  refresh_anchors: true
  refresh_day: "7"
  refresh_hour: "0"
```

Where, in each list item:

* **autobuild** is `true` to automatically build the environment whenever a pull request is created. `false` otherwise. Optional, defaults to `true`.
* **autodelete** is `true` to automatically delete the environment whenever a pull request is merged or closed. `false` otherwise. Optional, defaults to `true`.
* **autorebuild** is `true` to automatically rebuild the environment whenever the pull request updated. `false` otherwise. Optional, defaults to `true`.
* **build_timeout** is the timeout in seconds to wait for an environment to build before throwing an error. Optional, defaults to 1200 seconds (20 minutes).
* **envvars** is the list of environment variables to pass to containers in the environment. Each item must have a `name` and a `value` key. Optional.
* **domain** is the domain to use when provisioning the environment. Optional, if not specified, the project domain is used.
* **ip_allow** is a list of IP addresses which may access the environment. Optional.
* **ip_deny** is a list of IP addresses which may not access the environment Optional.
* **provider_deployment** is `true` to enable builds to add comments to the upstream merge request. `false` otherwise. Optional, defaults to `false`.
* **provider_forks** is `true` to enable environments to be built from forks of the merge request. Optional, defaults to `false`.
* **quota** is the disk quota in GB allowed for the environment. Optional, defaults to no limit.
* **rebuild_orphaned** is `true` to rebuild this environment when the base environment is rebuilt. Optional, defaults to `false`.
* **rebuild_stale** "is `true` to rebuild this environment when the base environment is refreshed. Optional, defaults to `false`.
* **refresh_anchors** is `true` to automatically refresh base previews in this repository when the `anchor` property is set to `true`. Optional, defaults to `true`.
* **refresh_day** the day of the week as an integer on which to refresh this environment. Sunday is 0, and Saturday is 6. Optional, defaults to 7 (everyday).
* **refresh_hour** is the hour in UTC 24 hour format on which to refresh the environment. Optional, defaults to 0 (midnight).

## Limitations

### Project names must be unique

The Tugboat API relies on internally generated IDs to update records. Since this role does not use IDs, it relies on the project name to determine a match when doing updates or deletions of repositories. If two or more projects exist which have the same name accessible with the same API key, the first is taken and the second ignored.

### Repository names must be unique within the project

Similar to the above limitation with project names, repository names must be unique within the project. If two or more repositories are found with the same name, the first is always taken, the second ignored.

### Unchanged repositories always update

Another limitation is that even if a repository has no changes from the last invocation, the role will still call the API to update the record. Since the update record is the same as the old record, no effective change has been made, but it does result in additional API calls to Tugboat.

## Dependencies

None.

## Example Playbook

```yaml
    ---
    - hosts: all
      vars:
        tugboat_api_token: '1234567890abscdefg'
        tugboat_repo:
          project: "my-clients"
          name: 'example.com'
          state: present
          autobuild: true
          autodelete: true
          autorebuild: true
          build_timeout: "1200"
          envvars:
            - name: ANSIBLE_VERBOSITY
              value: "4"

      tasks:
        - name: provision tugboat repo
          include_role:
            name: "ten7.tugboat_repo"
```

## Manipulating large envvars

In some cases, you may need to pass a large, complex value via environment variables such as FLIGHTDECK_CONFIG. In this case, you may set up an multi-line document in YAML, and then pass it as a variable to the role, performing whatever processing is necessary:

```yaml
    ---
    - hosts: all
      vars:
        tugboat_api_token: '1234567890abscdefg'
        flightdeck_config_example_com: |
          ---
          flightdeck_debug: yes
        tugboat_repo:
          project: "my-clients"
          name: 'example.com'
          state: present
          autobuild: true
          autodelete: true
          autorebuild: true
          build_timeout: "1200"
          envvars:
            - name: ANSIBLE_VERBOSITY
              value: "4"
            - name: FLIGHTDECK_CONFIG
              value: "{{ flightdeck_config_example_com | b64encode }}"
      tasks:
        - name: provision tugboat repo
          include_role:
            name: "ten7.tugboat_repo"
```

## Provisioning multiple domains

One limitation of this role is it can only be run on one domain at a time. This was done to make the role easier to read and maintain. Since the role is re-entrant, you can run it in a loop:

```yaml
    ---
    - hosts: all
      vars:
        tugboat_api_token: '1234567890abscdefg'
        tugboat_repos:
          - project: "my-clients"
            name: 'example.com'
            state: present
            autobuild: true
            autodelete: true
            autorebuild: true
            build_timeout: "1200"
            envvars:
              - name: ANSIBLE_VERBOSITY
                value: "4"

      tasks:
        - name: Provision repos
          include_tasks: 'tasks/main.yml'
          loop: "{{ tugboat_repos }}"
          loop_control:
            label: "{{ tugboat_repo.name }}"
            loop_var: tugboat_repo
```

## License

GPL v3

## Author Information

This role was created by [socketwench](https://deninet.com/) for [TEN7](https://ten7.com).
