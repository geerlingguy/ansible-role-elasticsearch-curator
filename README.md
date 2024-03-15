# Ansible Role: Elasticsearch Curator

[![CI](https://github.com/geerlingguy/ansible-role-elasticsearch-curator/workflows/CI/badge.svg?event=push)](https://github.com/geerlingguy/ansible-role-elasticsearch-curator/actions?query=workflow%3ACI)

An Ansible Role that installs [Elasticsearch Curator](https://github.com/elasticsearch/curator) on RedHat/CentOS or Debian/Ubuntu.

## Requirements

None, but it's a lot more helpful if you have Elasticsearch running somewhere!

On RedHat/CentOS, make sure you have the EPEL repository configured, so the `python-pip` package can be installed. You can install the EPEL repo by simply adding `geerlingguy.repo-epel` to your playbook's roles.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    elasticsearch_curator_version: ''

The version of `elasticsearch-curator` to install. Available versions are listed on the [Python Package Index](https://pypi.org/project/elasticsearch-curator/). By default, no version is specified, so the latest version will be installed.

    elasticsearch_curator_cron_jobs:
      - name: "Run elasticsearch curator actions."
        job: "/usr/local/bin/curator ~/.curator/action.yml"
        minute: 0
        hour: 1

A list of cron jobs. Typically you would run one cron job using the actions defined in `action.yml`, but you can split up cron jobs or use the `curator_cli` to run actions individually instead of via an action file. You can add any of `minute`, `hour`, `day`, `weekday`, and `month` to the cron jobs—values that are not explicitly set will default to `*`. You can also use `state` to define whether the job should be `present` or `absent`.

    elasticsearch_curator_config_directory: ~/.curator

The directory inside which Curator's configuration (and action file) will be stored.

    elasticsearch_curator_hosts:
      - 'localhost:9200'
    elasticsearch_curator_http_auth: ''

These variables control parameters used in the default `elasticsearch_curator_yaml`. If you define your own custom `elasticsearch_curator_yaml`, you may not need to define or override these variables. `_hosts` is a list of hosts with ports, and `_http_auth` is a basic `http_auth` `user:pass` string, if your Elasticsearch instance requires basic HTTP authorization.

    elasticsearch_curator_yaml: |
      ---
      client:
        hosts: {{ elasticsearch_curator_hosts | to_yaml }}
        url_prefix:
        use_ssl: False
        certificate:
        client_cert:
        client_key:
        ssl_no_validate: False
        http_auth: {{ elasticsearch_curator_http_auth }}
        timeout: 30
        master_only: False
      logging:
        loglevel: INFO
        logfile:
        logformat: default
        blacklist: ['elasticsearch', 'urllib3']

This YAML goes into the file `~/.curator/curator.yml` and stores the connection details Elasticsearch Curator uses to when connecting to Elasticsearch, as well as Curator logging configuration.

    elasticsearch_curator_action_yaml: |
      ---
      actions:
        1:
          action: delete_indices
          options:
            ignore_empty_list: True
            disable_action: False
          filters:
          - filtertype: pattern
            kind: prefix
            value: logstash-
            exclude:
          - filtertype: age
            source: name
            direction: older
            timestring: '%Y.%m.%d'
            unit: days
            unit_count: 45
            exclude:

This YAML goes into the file `~/.curator/action.yml` and defines the actions Curator performs when the default cron job is run. See documentation: [Curator actions file](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/actionfile.html).

    elasticsearch_curator_pip_package: 'python3-pip'
    elasticsearch_curator_pip_executable: "{{ 'pip3' if elasticsearch_curator_pip_package.startswith('python3') else 'pip' }}"

System pip package which needs to be installed, and the `pip` executable to run. For older OSes or when using Python 2, you may need to override this and change these to `python-pip`, and `pip`, respectively.

## Dependencies

  - geerlingguy.repo-epel (RedHat/CentOS only)

## Example Playbook

    - hosts: search
      roles:
        - { role: geerlingguy.elasticsearch-curator }

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
