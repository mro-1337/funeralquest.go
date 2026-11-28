# Running Funeral Quest behind Caddy

This guide is for sysops who put Caddy in front of `fq web` to get HTTPS. Caddy gets the certificate and renews it by itself.

**Seeing "connects out only" or "outbound only"?** Your town hasn't told other towns its public address. Do step 3 and it goes away.

## 1. Set up Funeral Quest

In the sysop pages (`/sysop`), open **Settings → Server**:

| Setting | Set it to |
|---|---|
| Web server port | `8080` |
| Web server listens on | `127.0.0.1` (only Caddy can reach the game directly) |
| Behind my own proxy | **Yes** |

Restart `fq web`.

## 2. Set up Caddy

Pick **one** of these Caddyfiles. Put your own domain in place of `fq.example.com` or `example.com`.

**Its own subdomain (recommended):** the game is at `https://fq.example.com/`.

```
fq.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

**A folder on your website:** the game is at `https://example.com/fq/`.

```
example.com {
    handle_path /fq/* {
        reverse_proxy 127.0.0.1:8080
    }
    # ... the rest of your site
}
```

Use `handle_path`, not `handle`.

Then run `caddy reload`.

For Caddy to get its certificate:

* Your domain (or subdomain) must point to your public IP address.
* Ports **80 and 443** must be forwarded on your router to the machine running Caddy.

## 3. Tell other towns your address

In **Settings → Network**, set **This server's web address** to the address you type in a browser:

* `https://fq.example.com`, or
* `https://example.com/fq`

Don't add `:8080` or a slash at the end. Save.

That's it. Linked towns pick up the new address the next time they hear from you, usually within a few minutes. The "outbound only" note then disappears.

## 4. Check it

* Open your address on a phone with Wi-Fi turned off. You should see the Funeral Quest log-on page.
* Ask a linked sysop to look at their **Network** tab. Your town should no longer say "outbound only".
* Or run this from another town's server:
  ```
  fq admin test-link https://fq.example.com
  ```
  It tests the connection step by step and says what's wrong. It may fail when you run it on your own machine, because some routers don't let you reach your own public address from inside.

## If it still says outbound only

| Problem | Fix |
|---|---|
| **This server's web address** is empty | Fill it in (step 3). |
| It says `http://localhost...` or has `:8080` | Use the public `https://` address instead. |
| The game page doesn't open from outside | Check the router (ports 80 and 443) and the DNS for your domain. |
| You used `handle /fq/*` | Change it to `handle_path /fq/*` and run `caddy reload`. |
| The folder in the address doesn't match the Caddyfile | Both must use the same folder (`/fq`). |
