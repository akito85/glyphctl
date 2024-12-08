Get Cloudflare GPG key

```bash
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
```

Set Cloudflare Repo

```bash
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```

Install Warp-CLI

```bash
sudo apt-get update && sudo apt-get install cloudflare-warp
```


### Initial Connection

To connect for the very first time:

1. Register the client `warp-cli registration new`.
2. Connect `warp-cli connect`.
3. Run `curl https://www.cloudflare.com/cdn-cgi/trace/` and verify that `warp=on`.

### Switching modes

You can use `warp-cli mode --help` to get a list of the modes to switch between. For example:

- **DNS only mode via DoH:** `warp-cli mode doh`.
- **WARP with DoH:** `warp-cli mode warp+doh`.

### Using 1.1.1.1 for Families

The Linux client supports all 1.1.1.1 for Families modes, in either WARP on DNS-only mode:

- **Families mode off:** `warp-cli dns families off`
- **Malware protection:** `warp-cli dns families malware`
- **Malware and adult content:** `warp-cli dns families full`