aws-overseer
============
Scheduled starting and stopping of EC2 instances with cron jobs.

If you are looking into this role then you might be trying to save some money by stopping non-critical EC2 instances during off hours. Why leave EC2 instances running if they are only used during specific time intervals and can be safely stopped? This role simply requires adding a couple tags to any instance that you want managed by a `start_stop_schedule`. It is also an experiment of using linux tools like `ansible`, `cron` and `at` for all functionality. There are some weird templated-out ansible playbooks if you examine further, but they work well.

The tags are:

- **`Overseer`**: the name of the overseer that manages this instance (set to `Ignore` to be ignored by `overseer`)
- **`StartStopSchedule`**: the name of the schedule to use for this instance

Instances are only stopped or started if a `start_stop_schedule` has been configured on the `overseer` host using this role. 

> Tip: Create an IAM user just for `overseer` so that you can use [CloudTrail](http://aws.amazon.com/cloudtrail/) to view all start-stop history

**Note:**

Example Playbook
----------------

```yaml
  ---
  - name: Configure the overseer host.
    hosts: overseer
    gather_facts: yes
    sudo: yes

    roles:
      - role: aws-overseer
        overseer_name: us-east-1-dev
        overseer_aws_region: us-east-1
        overseer_enforce: yes
        overseer_start_stop_schedules:
          - schedule_name: raleigh-business-hours
            start_cron_expression: "* 6 * * 1,2,3,4,5"
            stop_cron_expression: "* 18 * * 1,2,3,4,5"
            timezone: America/New_York
          - schedule_name: sao-paulo-business-hours
            start_cron_expression: "* 6 * * 1,2,3,4,5"
            stop_cron_expression: "* 18 * * 1,2,3,4,5"
            timezone: America/Sao_Paulo

      - role: aws-overseer
        overseer_name: us-west-2-dev
        overseer_aws_region: us-west-2
        overseer_start_stop_schedules:
          - schedule_name: san-francisco-business-hours
            start_cron_expression: "* 4 * * 1,2,3,4,5,6,7"
            stop_cron_expression: "* 20 * * 1,2,3,4,5,6,7"
            timezone: America/Los_Angeles
```

In this example the `us-east-1-dev` overseer will start and stop instances using two schedules:

- `raleigh-business-hours`: starts instances every M-F at 6 AM Raleigh time and stops them every M-F at 6 PM
- `sao-paulo-business-hours`: starts instances every M-F at 6 AM Sao Paulo time and stops them every M-F at 6PM

The `us-east-1-dev` overseer defaults to no enforcement, so instances that aren't tagged with an `Overseer` tag will keep on going, with no stops. The `us-west-2-dev` overseer will start and stop instances using one schedule:

- `san-francisco-business-hours`: starts instances every day of the week at 4 AM and stops them every day at 8 PM *<sup>where is the work-life balance</sup>*

...it does have enforcement turned on. Instances that don't have the `Overseer` tag will simply be stopped within the first billable hour they are started.

##### Enforcement

If `overseer_enforce` is turned on then a cron job will run every 10 minutes that will check for instances that do not have an `Overseer` tag and have been running for more than 50 minutes after launch. For each of the instances that the cron finds it will schedule stopping it 5 minutes in the future. The reason for the delay is that I intend on adding a notification based on tag in the future to give somebody time to fix their instance if need be.

> If previously enabled then the job is removed when disabled.

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
| overseer_enforce              | optional            | `no`                                     | stop any instances without an `Overseer` tag                  |
| overseer_start_stop_schedules | optional            | []                                       | an array of `start_stop_schedule` descriptions (below)        |

> An instance can be tagged with `Overseer=Ignore` to make it ignored by all `overseer` jobs, including enforcement.

**`start_stop_schedule` description**

| property              | required / optional | default | description
| ----------------------|---------------------|---------|---------------------------------------------------------------------------------------------------|
| schedule_name         | required            |         | the name to give this schedule                                                                    |
| start_cron_expression | required            |         | a [cron expression](http://en.wikipedia.org/wiki/Cron#CRON_expression) for the start instance job |
| stop_cron_expression  | required            |         | a [cron expression](http://en.wikipedia.org/wiki/Cron#CRON_expression) for the stop instance job  |
| timezone              | optional            | UTC     | the timezone to use for the overseer cron table                                                   |

> Any `start_stop_schedule` jobs are removed if they are not in the configuration. This makes sure old `start_stop_schedules` aren't going to accumulate over time in `/etc/cron.d`

Dependencies
------------

None

License
-------

[MIT](LICENSE)
