# firewall

Configures a basic firewall with [ufw](https://help.ubuntu.com/community/UFW): installs it, allows incoming traffic on the listed ports, sets default policies and enables it.

## Requirements

The role requires:

- the managed host to be Debian-based because it uses `apt`;
- privilege escalation with `become: true` when running;
- the `community.general` collection to be installed on the control node.

## Variables

All the defaults are defined in `defaults/main.yml`.

| Variable | Default | Description |
|---|---|---|
| `firewall_allowed_ports` | 22, 80, 443 | Ports to allow incoming traffic on. Each item has `port` and optional `proto` (defaults to `tcp`). |
| `firewall_default_incoming` | `deny` | Default policy for incoming traffic. |
| `firewall_default_outgoing` | `allow` | Default policy for outgoing traffic. |

Keep in mind that `firewall_allowed_ports` variable must include your SSH port. If you override the variable or omit it, enabling the firewall will cut off your SSH session and you will lose access to the host. If SSH listens on a non-standard port, put that port in the list instead of 22.

## Usage

```yaml
- hosts: all
  become: true
  roles:
    - role: firewall
      vars:
        firewall_allowed_ports:
          - { port: 22, proto: tcp }
          - { port: 8080, proto: tcp }
```

## Notes

ufw does not filter traffic to published Docker container ports. When a container is started with a published port, e.g. `-p 8080:80`, Docker adds its own iptables rules that send incoming traffic straight to the container, so it bypasses the ufw rules. The port stays reachable from outside even with `firewall_default_incoming: deny` and without a matching allow rule. `ufw status` does not list it either, which makes the exposure easy to miss.