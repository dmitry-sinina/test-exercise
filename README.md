# nginx in Podman, managed by Ansible

Ansible deploys nginx as a Podman Quadlet (systemd) service on the local host,
serving configurable content on a configurable port.

## Requirements

- Debian 13 (trixie) host, run locally (`inventory/hosts.yml` targets `localhost`)
- Ansible >= 2.15 (`apt install ansible` or `pip install ansible-core`)
- Root access via `su`: `ansible.cfg` uses `become_method = su`, so pass `-K`
  and enter the root password when prompted

Run all commands from the repository root so `ansible.cfg` is picked up.

## Install

1. Install Podman (one-time host preparation):

   ```sh
   ansible-playbook playbooks/site-prepare.yml -K
   ```

2. Deploy nginx:

   ```sh
   ansible-playbook playbooks/nginx.yml -K
   ```

   The playbook waits until the content is served and the container health check passes.

3. Verify:

   ```sh
   curl http://127.0.0.1:8080/
   ```

Both playbooks are idempotent. You can re-run them safely.

## Configuration

| Variable          | Default                                                  | Description                   |
|-------------------|----------------------------------------------------------|-------------------------------|
| `nginx_host_port` | `8080`                                                   | Host port nginx listens on    |
| `nginx_content`   | `Hello from nginx managed by Ansible + Podman Quadlet`    | Text served on `/`            |

Defaults are in `roles/nginx/defaults/main.yml`. Override them with `-e`:

```sh
ansible-playbook playbooks/nginx.yml -K -e nginx_host_port=9090 -e 'nginx_content="Hello v2"'
curl http://127.0.0.1:9090/
```

Changing either value and re-running the playbook updates the files and restarts the service.

## Destroy

1. Remove the nginx service, its data directory and its image:

   ```sh
   ansible-playbook playbooks/nginx-destroy.yml -K
   ```

2. Uninstall Podman and purge its data (optional). This step refuses to run while Quadlet services are still deployed:

   ```sh
   ansible-playbook playbooks/site-destroy.yml -K
   ```
