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
   PROXY_LOGIN: username
   PROXY_PASSWORD: password
   ```

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

> `http.noProxy` used for VSC hosts that use proxy but ignore proxy authorization and generates errors.  
> `http.proxyStrictSSL` can be disabled by setting it to **false** if the proxy server is not running.

```json
{
    "http.proxy": "socks5://${PROXY_LOGIN}:${PROXY_PASSWORD}@${ansible_host}$:${socks_external_port}",
    "http.proxyAuthorization": null,
    "http.proxyStrictSSL": true,
    "http.proxySupport": "fallback",
    "http.noProxy": [
        "avatars.githubusercontent.com",
        "default.exp-tas.com",
        "education.github.com",
        "img.shields.io",
        "main.vscode-cdn.net",
        "marketplace.visualstudio.com",
        "mechatroner.gallery.vsassets.io",
        "mechatroner.gallerycdn.vsassets.io",
        "mobile.events.data.microsoft.com",
        "update.code.visualstudio.com",
        "www.gravatar.com",
        "www.vscode-unpkg.net"
    ]
}
```

## Debug

Code below helps debugging 3proxy and show [error code](https://kraken-proxy.ru/en/page/kodyi-oshibok-v-logah-3proxy) for failed connection attempts.

```bash
docker logs proxy-3proxy-1 --since 1800s | grep -v "00000" \
   | jq -c '(.time_unix | tonumber | strftime("%H:%M:%S")) + " - " + .request.hostname + " - " + .error.code'

docker logs proxy-3proxy-1 --since 1800s | grep -v "00000" \
   | jq -c '.request.hostname + " - " + .error.code' | sort -u
```
