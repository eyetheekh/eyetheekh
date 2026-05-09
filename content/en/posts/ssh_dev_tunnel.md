---
title: "Remote SSH Tunnels for Development"
date: 2025-06-05T14:00:00+05:30
draft: false
tags: ["linux", "ssh"]
---

VS Code provides a simple interface for working with remote servers using the Remote SSH extension pack. It also has a built-in port forwarding feature that allows forwarding open ports from a remote server to the local machine. This was very useful since I could build and run apps on the remote server and access them through my machine's localhost. I really liked this workflow because it was convenient and highly useful during development.

However, there were also a few aspects I disliked, such as requiring authentication with GitHub/Microsoft, and more importantly, the latency. During development, that delay can become somewhat irritating.

I used this setup frequently and eventually wanted to understand how it actually worked.

It relies on Dev Tunnels (Microsoft-managed intermediary servers), which route local traffic through Microsoft's cloud infrastructure before reaching the user. There are also restrictions on usage limits, bandwidth, and the number of connections. Refer to the VS Code Docs: [https://code.visualstudio.com/docs/debugtest/port-forwarding](https://code.visualstudio.com/docs/debugtest/port-forwarding).

That was when I started looking for alternatives. I came across [Zed](https://zed.dev/docs/remote-development), as well as native SSH tunneling directly from the host. Zed was great and noticeably faster, but due to the lack of some extensions that I rely on in VS Code, I eventually settled on using native SSH tunnels instead. They turned out to be extremely lightweight and reliable.

After doing this repeatedly, I created a small shell script to automate tunnel creation. The script accepts multiple ports and forwards them automatically. I also added it to my `.bashrc` so I could use it easily from any terminal tab.

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configuration (hard-coded)

KEYFILE="/home/eyetheekh/Documents/dev_gpu.pem"
SSH_USER="dev"
HOST="103.182.259.248"

# Ensure at least one port is passed
if [ $# -lt 1 ]; then
  echo "Usage: $0 <port1> [port2 ...]"
  echo "Example: $0 8021 9001 6379"
  exit 1
fi

# Build `-L` arguments for each port
L_ARGS=()
for port in "$@"; do
  L_ARGS+=( -L "${port}:localhost:${port}" )
done

# SSH options
SSH_OPTS=(
  -i "${KEYFILE}"
  -o ExitOnForwardFailure=yes
  -o ServerAliveInterval=30
  -f
  -N
)

# Establish the tunnel
echo "Opening SSH tunnel to ${SSH_USER}@${HOST} for ports: $*"
ssh "${SSH_OPTS[@]}" "${L_ARGS[@]}" "${SSH_USER}@${HOST}"
echo "Tunnel established. Access your services on localhost ports: $*."
```

**Usage:**

```bash
./tunnel.sh 8000 6379 9001
```

This forwards:

| Local Port | Remote Port    |
| ----------- | ---------------- |
| 8000        | localhost:8000   |
| 6379        | localhost:6379   |
| 9001        | localhost:9001   |

After this:

- `localhost:8000` → remote API
- `localhost:6379` → remote Redis
- `localhost:9001` → remote inference server

Happy tunneling.