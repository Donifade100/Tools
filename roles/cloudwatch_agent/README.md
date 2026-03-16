# CloudWatch Agent Description

This role installs the Amazon CloudWatch Agent and configures it to publish:

- Metrics (CPU, memory, disk, network, processes)
- JVM metrics via collectd JMX (required for JVM metrics)
- Log files (JBoss/controller/access logs + selected OS logs)


## Requirements

No requirements. Just need a privileged user.

## Supported Platforms

- RHEL/CentOS 7/8/9
- Amazon Linux 2023
- Amazon Linux 2

## Role Variables

| Variable | Description | Default |
| --- | --- | --- |
| `cloudwatch_agent_install_method` | Install method for the CloudWatch Agent (`auto`, `yum`, `rpm`) | `auto` |
| `cloudwatch_agent_rpm_url` | RPM URL used when `install_method` is `rpm` or `auto` fallback | `https://s3.amazonaws.com/amazoncloudwatch-agent/redhat/amd64/latest/amazon-cloudwatch-agent.rpm` |
| `cloudwatch_agent_config_path` | Destination path for `amazon-cloudwatch-agent.json` | `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json` |
| `cloudwatch_agent_metrics_namespace` | CloudWatch Metrics namespace | `CORP/JBossEAP` |
| `cloudwatch_agent_collectd_service_address` | collectd UDP listener used by CloudWatch Agent | `udp://127.0.0.1:25826` |
| `cloudwatch_agent_log_collect_list` | List of log file objects for CW Agent to collect | (see `defaults/main.yml`) |
| `cloudwatch_agent_collectd_conf_path` | Path to collectd main config | `/etc/collectd.conf` |
| `cloudwatch_agent_collectd_conf_d_dir` | Directory for collectd drop-in configs | `/etc/collectd.d` |
| `cloudwatch_agent_collectd_jmx_conf_path` | Path for JMX config drop-in | `/etc/collectd.d/jmx.conf` |

## Dependencies

This role has no external dependencies.

## Example Playbooks

### Basic Installation

```yaml
- hosts: jboss_ec2
  become: true
  tasks:
    - name: Install CloudWatch Agent + collectd
      ansible.builtin.import_role:
        name: corp.jboss_eap.cloudwatch_agent
```

### Force RPM Install (unregistered RHEL)

```yaml
- hosts: jboss_ec2
  become: true
  tasks:
    - name: Install CloudWatch Agent via RPM
      ansible.builtin.import_role:
        name: enlyte.tools.cloudwatch_agent
      vars:
        cloudwatch_agent_install_method: rpm
        cloudwatch_agent_metrics_namespace: MyNameSpace
```

### Override Logs (example)

```yaml
- hosts: jboss_ec2
  become: true
  tasks:
    - name: Install CloudWatch Agent with custom logs
      ansible.builtin.import_role:
        name: corp.jboss_eap.cloudwatch_agent
      vars:
        cloudwatch_agent_log_collect_list:
          - file_path: /weblogs/jbc/n*/log/server.log*
            log_group_name: /jboss/serverlog
            log_stream_name: "{instance_id}-{hostname}"
          - file_path: /var/log/messages
            log_group_name: /linux/messages
            log_stream_name: "{instance_id}"
```

## Security Considerations

- This role configures JMX collection via collectd using a localhost JMX URL. You must ensure JBoss JMX is enabled safely.
- CloudWatch Agent requires an IAM role with permissions to publish metrics/logs (for example, `CloudWatchAgentServerPolicy`), and your log groups must be permitted by policy.

## License

Internal use only.

## Author Information

Mola Onifade donifade15@gmail.com

## Changelog

- v0.1.0 (YYYY-MM-DD): Initial release

## Contributing

Please refer to your internal contribution guidelines for this project.
