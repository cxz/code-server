<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
# FAQ

- [Questions?](#questions)
- [What's the deal with extensions?](#whats-the-deal-with-extensions)
- [Where are extensions stored?](#where-are-extensions-stored)
- [How is this different from VS Code Codespaces?](#how-is-this-different-from-vs-code-codespaces)
- [How should I expose code-server to the internet?](#how-should-i-expose-code-server-to-the-internet)
- [How do I securely access web services?](#how-do-i-securely-access-web-services)
  - [Sub-domains](#sub-domains)
  - [Sub-paths](#sub-paths)
- [Multi-tenancy](#multi-tenancy)
- [Docker in code-server docker container?](#docker-in-code-server-docker-container)
- [Collaboration](#collaboration)
- [How can I disable telemetry?](#how-can-i-disable-telemetry)
- [How does code-server decide what workspace or folder to open?](#how-does-code-server-decide-what-workspace-or-folder-to-open)
- [How do I debug issues with code-server?](#how-do-i-debug-issues-with-code-server)
- [Heartbeat File](#heartbeat-file)
- [How does the config file work?](#how-does-the-config-file-work)
- [Enterprise](#enterprise)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Questions?

Please file all questions and support requests at https://www.reddit.com/r/codeserver/.

The issue tracker is **only** for bugs.

## What's the deal with extensions?

Unfortunately, the Microsoft VS Code Marketplace license prohibits use with any non Microsoft
product.

See https://cdn.vsassets.io/v/M146_20190123.39/_content/Microsoft-Visual-Studio-Marketplace-Terms-of-Use.pdf

> Marketplace Offerings are intended for use only with Visual Studio Products and Services
> and you may only install and use Marketplace Offerings with Visual Studio Products and Services.

As a result, Coder has created its own marketplace for open source extensions. It works by scraping
GitHub for VS Code extensions and building them. It's not perfect but getting better by the day with
more and more extensions.

Issue [#1299](https://github.com/cdr/code-server/issues/1299) is a big one in making the experience here
better by allowing the community to submit extensions and repos to avoid waiting until the scraper finds
an extension.

If an extension is not available or does not work, you can grab its VSIX from its Github releases or
build it yourself. Then run the `Extensions: Install from VSIX` command in the Command Palette and
point to the .vsix file.

See below for installing an extension from the cli.

Feel free to file an issue to add a missing extension to the marketplace.

If you have your own custom marketplace, it is possible to point code-server to it by setting
`$SERVICE_URL` and `$ITEM_URL` to point to it.

## Where are extensions stored?

Defaults to `~/.local/share/code-server/extensions`.

If the `XDG_DATA_HOME` environment variable is set the data directory will be
`$XDG_DATA_HOME/code-server/extensions`. In general we try to follow the XDG directory spec.

You can install an extension on the CLI with:

```bash
# From the Coder extension marketplace
code-server --install-extension ms-python.python

# From a downloaded VSIX on the file system
code-server --install-extension downloaded-ms-python.python.vsix
```

## How is this different from VS Code Codespaces?

VS Code Codespaces is a closed source and paid service by Microsoft. It also allows you to access
VS Code via the browser.

However, code-server is free, open source and can be ran on any machine without any limitations.

While you can self host environments with VS Code Codespaces, you still need to an Azure billing
account and you access VS Code via the Codespaces web dashboard instead of directly connecting to
your instance.

## How should I expose code-server to the internet?

Please follow [./guide.md](./guide.md) for our recommendations on setting up and using code-server.

code-server only supports password authentication natively.

**note**: code-server will rate limit password authentication attempts at 2 a minute and 12 an hour.

If you want to use external authentication (i.e sign in with Google) you should handle this
with a reverse proxy using something like [oauth2_proxy](https://github.com/pusher/oauth2_proxy).

For HTTPS, you can use a self signed certificate by passing in just `--cert` or
pass in an existing certificate by providing the path to `--cert` and the path to
its key with `--cert-key`.

If `code-server` has been passed a certificate it will also respond to HTTPS
requests and will redirect all HTTP requests to HTTPS. Otherwise it will respond
only to HTTP requests.

You can use [Let's Encrypt](https://letsencrypt.org/) to get an SSL certificate
for free.

Again, Please follow [./guide.md](./guide.md) for our recommendations on setting up and using code-server.

## How do I securely access web services?

code-server is capable of proxying to any port using either a subdomain or a
subpath which means you can securely access these services using code-server's
built-in authentication.

### Sub-domains

You will need a DNS entry that points to your server for each port you want to
access. You can either set up a wildcard DNS entry for `*.<domain>` if your domain
name registrar supports it or you can create one for every port you want to
access (`3000.<domain>`, `8080.<domain>`, etc).

You should also set up TLS certificates for these subdomains, either using a
wildcard certificate for `*.<domain>` or individual certificates for each port.

Start code-server with the `--proxy-domain` flag set to your domain.

```
code-server --proxy-domain <domain>
```

Now you can browse to `<port>.<domain>`. Note that this uses the host header so
ensure your reverse proxy forwards that information if you are using one.

### Sub-paths

Just browse to `/proxy/<port>/`.

## Multi-tenancy

If you want to run multiple code-server's on shared infrastructure, we recommend using virtual
machines with a VM per user. This will easily allow users to run a docker daemon. If you want
to use kubernetes, you'll definitely want to use [kubevirt](https://kubevirt.io) to give each
user a virtual machine instead of just a container. Docker in docker while supported requires
privileged containers which are a security risk in a multi tenant infrastructure.

## Docker in code-server docker container?

If you'd like to access docker inside of code-server, we'd recommend running a docker:dind container
and mounting in a directory to share between dind and the code-server container at /var/run. After, install
the docker CLI in the code-server container and you should be able to access the daemon as the socket
will be shared at /var/run/docker.sock.

In order to make volume mounts work, mount the home directory in the code-server container and the
dind container at the same path. i.e you'd volume mount a directory from the host to `/home/coder`
on both. This will allow any volume mounts in the home directory to work. Similar process
to make volume mounts in any other directory work.

## Collaboration

We understand the high demand but the team is swamped right now.

You can follow progress at [#33](https://github.com/cdr/code-server/issues/33).

## How can I disable telemetry?

Use the `--disable-telemetry` flag to completely disable telemetry. We use the
data collected only to improve code-server.

## How does code-server decide what workspace or folder to open?

code-server tries the following in order:

1. The `workspace` query parameter.
2. The `folder` query parameter.
3. The workspace or directory passed on the command line.
4. The last opened workspace or directory.

## How do I debug issues with code-server?

First run code-server with at least `debug` logging (or `trace` to be really
thorough) by setting the `--log` flag or the `LOG_LEVEL` environment variable.
`-vvv` and `--verbose` are aliases for `--log trace`.

```
code-server --log debug
```

Once this is done, replicate the issue you're having then collect logging
information from the following places:

1. stdout
2. The most recently created directory in the `~/.local/share/code-server/logs` directory.
3. The browser console and network tabs.

Additionally, collecting core dumps (you may need to enable them first) if
code-server crashes can be helpful.

## Heartbeat File

`code-server` touches `~/.local/share/code-server/heartbeat` once a minute as long
as there is an active browser connection.

If you want to shutdown `code-server` if there hasn't been an active connection in X minutes
you can do so by continuously checking the last modified time on the heartbeat file and if it is
older than X minutes, you should kill `code-server`.

[#1636](https://github.com/cdr/code-server/issues/1636) will make the experience here better.

## How does the config file work?

When `code-server` starts up, it creates a default config file in `~/.config/code-server/config.yaml` that looks
like this:

```yaml
bind-addr: 127.0.0.1:8080
auth: password
password: mewkmdasosafuio3422 # This is randomly generated for each config.yaml
cert: false
```

Each key in the file maps directly to a `code-server` flag. Run `code-server --help` to see
a listing of all the flags.

The default config here says to listen on the loopback IP port 8080, enable password authorization
and no TLS. Any flags passed to `code-server` will take priority over the config file.

The `--config` flag or `$CODE_SERVER_CONFIG` can be used to change the config file's location.

The default location also respects `$XDG_CONFIG_HOME`.

## Enterprise

Visit [our enterprise page](https://coder.com) for more information about our
enterprise offerings.
