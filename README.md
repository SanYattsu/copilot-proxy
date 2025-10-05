# Proxy Server Deployment with Ansible

This Ansible playbook automates the deployment of a proxy server using 3proxy and Docker Compose.

## Prerequisites

1. Ansible installed on your local machine.
2. SSH access to the target VPS.
3. Linux distribution - **Ubuntu** or you need to manually update docker installation vars.

## Configuration

1. Create strong vault password to encrypt secrets:

   ```bash
   openssl rand -base64 128 | tr -d '\n' > .vault
   ```

2. Edit `roles/proxy/files/.env` to set your credentials and encrypt them:

   ```bash
   ansible-vault encrypt roles/proxy/files/.env
   ```

3. Change VPS params in `inventory/main.yml`
4. Customize other variables in `roles/proxy/vars/main.yml` if needed. Don't forget to change **allowed_network** or you would be blocked to access the VM.

## Deployment

Run the playbook:

   ```bash
   ansible-playbook playbook.yml
   ```

## Accessing the Proxy

After deployment, the proxy will be available on SOCKS5 port 443 (TCP/UDP). You can test it with curl:

```bash
curl --socks5-hostname ${ansible_host}$:${socks_external_port} -U ${PROXY_LOGIN}:${PROXY_PASSWORD} check-host.net/ip
```

## Add the proxy to Visual Studio Code

1. Open VS Code.
2. Navigate to `File > Preferences > Settings` (or use the shortcut `Ctrl+,`).
3. In the Settings search bar, type **proxy**.
4. Locate the `Application > Proxy` section.
5. Open `setting.json` and edit proxy setting. Replace env vars with actual params.

```json
{
    "http.proxy": "socks5://${PROXY_LOGIN}:${PROXY_PASSWORD}@${ansible_host}$:${socks_external_port}",
    "http.proxyAuthorization": null,
    "http.proxyStrictSSL": false,
}
```
