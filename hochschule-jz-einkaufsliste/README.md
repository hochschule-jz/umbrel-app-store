# Einkaufsliste

The shared shopping list for one household, as an app in this store (`hochschule-jz-einkaufsliste`).
umbrelOS pulls its image from `ghcr.io/hochschule-jz/einkaufsliste`; the app's source is a separate, private project.

## First release

1. **Push the image.** Sign in to your registry (`docker login ghcr.io`), then from the root of the app's project, where this repository is checked out as `umbrel-store/`:

   ```bash
   ./release.sh ghcr.io/hochschule-jz/einkaufsliste
   ```

   It builds for the Raspberry Pi (arm64), pushes, and writes the pinned image and the version into
   this folder. Make the image public in the registry; umbrelOS pulls without signing in.

2. **Commit and push this repository**, `github.com/hochschule-jz/umbrel-app-store`, which the manifest's links and icon point to.

3. **Add the store in umbrelOS:** App Store, the menu at the top right, Community App Stores, paste the
   repository URL. Then install Einkaufsliste from „Julian's Apps“.

4. Open `http://umbrel.local:8642` (or your Tailscale name with that port) and create the first account.

## Updates

Raise `version` in `server/package.json` and `web/package.json`, run `release.sh` again, write what is
new into `releaseNotes` in `umbrel-app.yml`, and push this repository. umbrelOS then offers the update.

## Using it from anywhere, also without a connection (https)

Over `http://umbrel.local:8642` the app only works at home and needs the server for every start. With an
https address it can be installed on the phone, starts without a connection, and the Siri shortcut
accepts it. The Umbrel is reached through Tailscale, so nothing is opened to the internet.

There are two ways. Both need Tailscale on the Umbrel and on every device.

### A: your own domain (works on every phone)

A name under your own domain is in the public DNS, so it also resolves on phones whose DNS is fixed by a
company profile, where Tailscale's own names do not. The name points at the Umbrel's Tailscale address,
which only your own devices can reach. Below, `einkauf.example.eu` stands for your name and `100.x.y.z`
for the Umbrel's Tailscale address (shown in the Tailscale app).

1. **DNS record.** At the domain's DNS provider add an `A` record for the name with the value `100.x.y.z`.
2. **Check before anything else.** On each phone, with Tailscale on, open `http://einkauf.example.eu:8642`.
   The sign-in page must load. If a phone says the server cannot be found, its DNS refuses private
   addresses and this way does not work for it. (Some endings, like `.dev` or `.app`, only allow https;
   there a certificate warning at `https://…:8642` shows the same thing.)
3. **API key.** Create a key at the DNS provider that may edit DNS records for this domain only. It lets
   the proxy prove to Let's Encrypt that the domain is yours; the Umbrel itself is not reachable for that.
4. **Proxy.** Install „Nginx Proxy Manager“ from the Umbrel App Store. On Umbrel it serves https on port
   `40443`. Ignore its note about port forwarding on the router; that is for public access.
5. **Certificate.** In Nginx Proxy Manager: Certificates › Add › „Let's Encrypt via DNS“. Enter the name,
   choose your DNS provider, put the key into the credentials box in place of the placeholders, and set
   120 propagation seconds. It renews by itself.
6. **Proxy host.** Hosts › Proxy Hosts › Add: the name, scheme `http`, forward to `100.x.y.z` port `8642`,
   „Block Common Exploits“ on. On the SSL tab choose the certificate and turn on „Force SSL“ and
   „HTTP/2 Support“. Behind the gear icon, add this line, or changes from other devices arrive late:

   ```
   proxy_buffering off;
   ```

7. **Phones.** Open `https://einkauf.example.eu:40443`, sign in, add it to the Home Screen and delete an
   older icon. Set up the Siri shortcut from there (Einstellungen › Siri-Kurzbefehl), so it gets this address. Turn on
   „VPN On Demand“ in the Tailscale app, so the phone stays connected away from home.

### B: Tailscale Serve (no domain needed)

Simpler, but the address is a Tailscale name, which does not resolve on phones with a DNS profile.

1. In the Tailscale admin console, on the DNS page, turn on MagicDNS and „HTTPS Certificates“. This puts
   the Umbrel's machine name and the tailnet name into a public certificate log.
2. On the Umbrel, in a terminal:

   ```bash
   sudo docker exec tailscale_web_1 tailscale serve --bg --https=8443 http://localhost:8642
   ```

   ```bash
   sudo docker exec tailscale_web_1 tailscale serve status
   ```

   The second command shows the address, like `https://umbrel.tail1234.ts.net:8443`. If the container
   has another name, `sudo docker ps --format '{{.Names}}' | grep -i tailscale` shows it.
3. Use that address on the phones as in step 7 above.

To undo: `sudo docker exec tailscale_web_1 tailscale serve --https=8443 off`. If the address stops
answering after an update of Umbrel's Tailscale app, run the `serve` command again.

### Either way

- Check: open the app from the new icon, switch to flight mode, close the app and open it again. The lists
  appear with the „Offline“ row; changes are sent once the Umbrel is in reach again.
- The address only answers while Tailscale is on, at home too. With Tailscale off the app works from its
  copy on the phone and sends the changes later.
- A new address means signing in again on each device.
- Updates arrive by themselves: the app fetches a new version when it is opened and reloads once.
- Do not use Tailscale Funnel or a port forward on the router: both put the app on the open internet.

## Notes

- Data (database and photos) lives in the app's data folder on the Umbrel and survives updates.
- Umbrel's own login is not put in front of the app (`PROXY_AUTH_ADD: "false"`): the app has its own
  sign-in, and the Siri shortcut has to reach it.
- `port: 8642` must not be used by another app on the Umbrel; change it in `umbrel-app.yml` if it is.
- Never change the app `id` after the first install; the data folder is named after it.
