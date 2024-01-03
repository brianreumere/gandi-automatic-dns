**This project is no longer under active development**, but I will do my best to address any bug reports. See [porkbun-automatic-dns](https://github.com/brianreumere/porkbun-automatic-dns) for its spiritual successor for a different domain/DNS provider. Thanks to everyone who has contributed by reporting issues and opening pull requests over the 11 years (!) since I started this project.

gad
===

This script is intended to be used as a cron job to maintain the accuracy of multiple A or AAAA records in a Gandi.net zonefile. External IP address discovery is done via a network interface, [OpenDNS](http://www.opendns.com), or a custom command piped to standard input. The result is compared to the value in the active version of the zonefile of each record in `RECORD-NAMES`.

Prerequisites
=============

For Gandi's LiveDNS platform, generate a personal access token (PAT) to authenticate. [API keys are deprecated](https://api.gandi.net/docs/changelog/) and are not supported by `gad` (if you'd like to fully migrate to PATs, you should delete any API keys from the authentication options of your [account settings](https://account.gandi.net/en)). Generate a PAT by selecting your organization from the [Organizations panel](https://admin.gandi.net/organizations/) and clicking the button to create a token in the Personal Access Token (PAT) section. You should restrict the token to the domain names that you plan on updating with `gad`, and only select the `See and renew domain names` and `Manage domain name technical configurations` permissions.

[If you are on Gandi's legacy platform, request an API key here](https://www.gandi.net/admin/apixml/). Your domain also needs to be using a zone that you are allowed to edit. The default Gandi zone does not allow editing, so you must create a copy.

Requirements
============

  * Bourne shell
  * OpenSSL or [LibreSSL](http://www.libressl.org)

If you're using an OS that doesn't include the `ifconfig` or `dig` commands you have three options:
  * Install a package that provides the `dig` command, commonly `bind-tools` or `dnsutils` (to use IP discovery via OpenDNS)
  * Install a package that provides the `ifconfig` command, commonly `net-tools` (to use IP discovery via a network interface)
  * Use the `-s` flag and pipe a custom command that outputs your external IP address to `gad`, e.g., ```curl ipinfo.io/ip | gad -s -a APIKEY -d EXAMPLE.COM -r "RECORD-NAMES"```

Installation
============

The simplest way to install `gad` is to clone this repository. You can optionally add the repository folder to your `PATH` environment variable to make running `gad` easier, but you should specify its full path when creating a crontab entry. Personally, I have a `~/bin` folder that is already in my `PATH` where I create a symlink for `gad`, and a `~/git` folder that I clone the repository into:

```
git clone https://github.com/brianreumere/gandi-automatic-dns.git ~/git/gandi-automatic-dns
ln -s /home/brian/git/gandi-automatic-dns/gad /home/brian/bin/gad
```

To set up a crontab entry to run `gad` on a schedule, store your API key in the `~/.gandiapi` file (run `chmod 600 ~/.gandiapi` to make sure no other users have permissions to this file), and then run `crontab -e` and add a line similar to the following (this example will run `gad` every 15 minutes and update the `@` and `www` records of the domain `example.net`):

```
0,15,30,45 * * * * /home/brian/bin/gad -i em0 -d example.net -r "@ www"
```

If you have [issues with the `openssl s_client` command hanging](https://github.com/brianreumere/gandi-automatic-dns/issues/31), and you have access to the `timeout` utility from GNU coreutils, you can precede the path to the `gad` command with something like `timeout -k 10s 60s` (to send a `TERM` signal after 60 seconds and a `KILL` signal 10 seconds later if it's still running).

The v5/LiveDNS API [allows up to 1000 requests per minute](https://api.gandi.net/docs/reference/) and [the legacy API has a limit of 30 requests every 2 seconds](https://docs.gandi.net/en/reseller/faq/index.html), so you should be able to run `gad` more frequently than every 15 minutes (when no record updates are required, `gad` makes one API call per record, and one additional call when using the legacy API to get the active zone ID).

Command-line usage
==================

Run `gad` with no options or `gad -h` to view this usage info from the command line.

```
Usage: gad [-h] [-x] [-6] [-f] [-t] [-e] [-v] [-s] [-i EXT_IF] [-p KEYFILE|-a APIKEY] [-l TTL] -d EXAMPLE.COM -r "RECORD-NAMES"

-h: Print this usage info and exit
-x: Use Gandi's legacy XML-RPC API
-6: Update AAAA record(s) instead of A record(s)
-f: Force the creation of a new zonefile regardless of IP address or TTL discrepancy
-t: On Gandi's legacy DNS platform, if a new version of the zonefile is created, don't activate it; on LiveDNS, just print the updates that would be made if this flag wasn't used.
-e: Print debugging information to stdout
-v: Print information to stdout even if an update isn't needed
-s: Use stdin instead of OpenDNS to determine external IP address

-i EXT_IF: The name of your external network interface (optional, if provided uses ifconfig instead of OpenDNS to determine external IP address
-p KEYFILE: Path to the file that contains your PAT or API key (defaults to ~/.gandiapi)
-a APIKEY: Your PAT (for LiveDNS) or API key (for the legacy API) provided by Gandi (optional, loaded from a file if not specified)
-l TTL: Set a custom TTL on records (optional, and only supported on LiveDNS)
-d EXAMPLE.COM: The domain name whose active zonefile will be updated (required)
-r RECORD-NAMES: A space-separated list of the name(s) of the A or AAAA record(s) to update or create (required)
```

Function syntax
============

```
rpc "methodName" "datatype" "value" "struct" "name" "datatype" "value"
```

The `rpc` function can accept an arbitrary number of datatype/value pairs and structs and their member name/datatype/value tuples. structs _must_ be last! Valid method names can be found in the [legacy XML-RPC API documentation](http://doc.rpc.gandi.net/index.html). Note that the `APIKEY` value from the command line is automatically included as the first parameter.

```
rest "verb" "apiEndpoint" "body"
```

The `rest` function can call arbitrary endpoints of Gandi's LiveDNS REST API. If the verb is not `GET`, the function expects a third parameter to use as the body of the `POST` or `PUT` request. Valid API endpoints can be found in the [LiveDNS API documentation](https://api.gandi.net/docs/livedns/). Your PAT from the command line, the file specified by the `-p` flag, or the `~/.gandiapi` file is automatically included in the `Authorization` header.
