---
description: >-
  In this guide, we’ll explore how to configure Docker remote API access over
  the Tailscale network using Windows netsh commands.
---

# 🐋 Allow Docker Remote API access over Tailscale network

## Security Considerations

While this method allows for convenient access to the Docker remote API, it comes with potential security risks. Exposing the Docker API without proper security measures can lead to unauthorized access, allowing anyone with the correct IP address to execute commands on your Docker daemon.

Therefore, this setup should only be used in trusted environments, such as your known Tailscale network. Additionally, consider implementing access policy restrictions within Tailscale to control which devices can access your Docker API.

It’s crucial to ensure that only authorized devices have the ability to connect, thus minimizing the risk of exposure to unauthorized users.

## Steps

### Get Tailscale IP

Either check via Tailscale menu in Windows tray icon or execute the following command in Powershell:

```powershell
tailscale ip
```

#### Enable Docker Remote API on localhost

* Open **Docker Desktop**, go to **Settings** → **General**, and ensure the option **“Expose daemon on tcp://localhost:2375 without TLS”** is checked.
* Click **Apply & Restart**.

### Create Port Proxy Using netsh

```
netsh interface portproxy add v4tov4 listenaddress=[Tailscale-Internal-IP] listenport=2375 connectaddress=127.0.0.1 connectport=2375
```

Replace \[Tailscale-Internal-IP] with the actual IP address you retrieved earlier from tailscale ip.

### Verify the Configuration

```
netsh interface portproxy show all
```

### Test Docker API Access

```
curl http://[Tailscale-Internal-IP]:2375/info
```
