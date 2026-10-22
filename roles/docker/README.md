# docker

Installs Docker Engine from the official Docker apt repository, starts the Docker service and enables it at boot.

## Requirements

The role requires:

- the managed host to be Ubuntu or Debian, because it uses `apt` and the Docker repository path is built from the distribution name;
- privilege escalation with `become: true` when running;
- fact gathering to be enabled, because the repository URL and release name are taken from the host facts;
- the managed host to have access to `download.docker.com`.

## Variables

All the defaults are defined in `defaults/main.yml`.

| Variable | Default | Description |
|---|---|---|
| `docker_packages` | `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin` | Packages to install from the Docker repository. |

## Usage

```yaml
- hosts: all
  become: true
  roles:
    - docker
```

If you do not use Docker Compose or do not build images on your hosts, you can install only the packages you need. Overriding the variable replaces the whole list, so the example below omits `docker-compose-plugin` and `docker-buildx-plugin`:

```yaml
- hosts: all
  become: true
  roles:
    - role: docker
      vars:
        docker_packages:
          - docker-ce
          - docker-ce-cli
          - containerd.io
```

## Notes

The role does not add users to the `docker` group. Users outside this group have to run Docker commands with `sudo`, otherwise access to the Docker socket is denied.

If packages that conflict with Docker Engine, e.g. `docker.io` or `podman-docker`, are already installed, the installation may fail. Remove them before running the role.

Docker modifies iptables directly, so published container ports are reachable from outside even if a firewall does not allow them. To keep a container port private, bind it to the loopback interface, e.g. `-p 127.0.0.1:8080:80`.