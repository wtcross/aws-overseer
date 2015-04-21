aws-overseer
============
Scheduled starting and stopping of EC2 instances with cron jobs.

If you are using this role then you are probably trying to save some money. Why leave EC2 instances running if they are only used during specific intervals and can be safely stopped?

Requirements
------------

The host operating system must be one of the following:

- Debian
- Ubuntu
- CentOS
- Red Hat Enterprise Linux
- Amazon Linux

Role Variables
--------------

| variable                      | required / optional | default                                  | description                                                   |
| ------------------------------|---------------------|------------------------------------------|-------------------------------------------------------------- |
| overseer_aws_region           | required            |                                          | the AWS region this overseer works in                         |
| overseer_aws_access_key       | required            |                                          | the AWS access key for the aws account this overseer works in |
| overseer_aws_secret_key       | required            |                                          | the AWS secret key for the aws account this overseer works in |
| overseer_home                 | optional            | `/opt/overseer`                          | the path of the home directory for overseer configuration     |
| overseer_name                 | optional            | `Default`                                | the name to give this overseer                                |
| overseer_start_stop_schedules | optional            | []                                       | an array of `start_stop_schedule` descriptions (below)        |

**`start_stop_schedule` description**

| property              | required / optional | default | description
| ----------------------|---------------------|---------|---------------------------------------------------|
| schedule_name         | required            |         | the name to give this schedule                    |
| start_cron_expression | required            |         | a cron expression for the start instance job      |
| stop_cron_expression  | required            |         | a cron expression for the stop instance job       |
| timezone              | optional            | UTC     | the timezone to use for the overseer cron table   |

Dependencies
------------

None

Example Playbook
----------------

```yaml
  ---
  - name: Configure all overseer hosts.
    hosts: overseer
    gather_facts: yes
    sudo: yes

    roles:
      - role: aws-overseer
        overseer_name: development
        overseer_aws_region: us-east-2
        overseer_start_stop_schedules:
          - schedule_name: raleigh-business-hours
            start_cron_expression: "* 6 * * 1,2,3,4,5"
            stop_cron_expression: "* 18 * * 1,2,3,4,5"
            timezone: America/New_York
          - schedule_name: sao-paulo-business-hours
            start_cron_expression: "* 6 * * 1,2,3,4,5"
            stop_cron_expression: "* 18 * * 1,2,3,4,5"
            timezone: America/Sao_Paulo
```

**Note**: Current configuration only allows for one `aws-overseer` per host.

License
-------

[MIT](LICENSE)
