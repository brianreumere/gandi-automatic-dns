gad
===

This script is intended to be used as a cron job to maintain the accuracy of multiple A or AAAA records in a Gandi.net zonefile. External IP address discovery is done via a network interface, [OpenDNS](http://www.opendns.com), or a custom command piped to standard input. The result is compared to the value in the active version of the zonefile of each record in `RECORD-NAMES`.

Prerequisites
=============

If you are on Gandi's legacy DNS platform, your domain needs to be using a zone that you are allowed to edit. The default Gandi zone does not allow editing, so you must create a copy. [There are instructions on Gandi's wiki to create an editable zone](http://wiki.gandi.net/en/dns/zone/edit). You only need to perform the first two steps. [There is a FAQ regarding this here](http://wiki.gandi.net/en/dns/faq#cannot_change_zone_file).

If you are on Gandi's newer v5/LiveDNS platform, you do not need to perform these steps.

[For Gandi's legacy platform, request an API key here](https://www.gandi.net/admin/apixml/). For the new LiveDNS platform, [API keys are deprecated](https://api.gandi.net/docs/changelog/) in favor of personal access tokens (PATs). Generate a PAT by selecting your organization from the [Organizations panel](https://admin.gandi.net/organizations/) and clicking the button to create a token in the Personal Access Token (PAT) section. You should restrict the token to the domain name that you plan on updating with `gad`, and only select the `See and renew domain names` and `Manage domain name technical configurations` permissions.

Requirements
============

  * Bourne shell
  * OpenSSL or [LibreSSL](http://www.libressl.org)

If you're using an OS that doesn't include the `ifconfig` or `dig` commands you have three options:
  * Install a package that provides the `dig` command, commonly bind-tools or dnsutils (to use IP discovery via OpenDNS)
  * Install a package that provides the `ifconfig` command, commonly net-tools (to use IP discovery via a network interface)
  * Use the `-s` flag and pipe a custom command that outputs your external IP address to `gad`, e.g., ```curl ipinfo.io/ip | gad -s -a APIKEY -d EXAMPLE.COM -r "RECORD-NAMES"```

Installation
============

The simplest way to install `gad` is to clone this repository. You can optionally add the repository folder to your `PATH` environment variable to make running `gad` easier, but you should specify its full path when creating a crontab entry. Personally, I have a `~/bin` folder that is already in my `PATH` where I create a symlink for `gad`, and a `~/git` folder that I clone the repository into:

```
git clone https://github.com/brianreumere/gandi-automatic-dns.git ~/git/gandi-automatic-dns
ln -s /home/brian/git/gandi-automatic-dns/gad /home/brian/bin/gad
```

To set up a crontab entry to run `gad` on a schedule, store your API key in the file `~/.gandiapi` (run `chmod 600 ~/.gandiapi` to make sure no other users have permissions to this file), and then run `crontab -e` and add a line similar to the following (this example will run `gad` every 15 minutes and update the `@` and `www` records of your domain):

```
0,15,30,45 * * * * /home/brian/bin/gad -5 -i em0 -d EXAMPLE.COM -r "@ www"
```

If you have [issues with the `openssl s_client` command hanging](https://github.com/brianreumere/gandi-automatic-dns/issues/31), and you have access to the `timeout` utility from coreutils, you can precede the path to the `gad` command with something like `timeout -k 10s 60s` (to send a `TERM` signal after 60 seconds and a `KILL` signal 10 seconds later if it's still running).

The v5/LiveDNS API [allows up to 1000 requests per minute](https://api.gandi.net/docs/reference/) and [the legacy API has a limit of 30 requests every 2 seconds, documented here](https://docs.gandi.net/en/reseller/faq/index.html), so you should be able to run `gad` more frequently than every 15 minutes (`gad` makes a maximum of 5 API calls to update a single DNS record: 1 API call to get the active zonefile for the domain, 1 API call per record to check its accuracy, 1 API call to create a new version of the zonefile if needed, 1 API call per record to update or create it if needed, and 1 API call to activate the new version of the zonefile).

See the next section for more information about command-line options.

Command-line usage
==================

Run `gad` with no options or `gad -h` to view usage info.

Function syntax
============

```
rpc "methodName" "datatype" "value" "struct" "name" "datatype" "value"
```

The `rpc` function can accept an arbitrary number of datatype/value pairs and structs and their member name/datatype/value tuples. structs _must_ be last! Valid method names can be found in the [Gandi API documentation](http://doc.rpc.gandi.net/index.html). Note that the APIKEY value from the command line is automatically included as the first parameter.

```
rest "verb" "apiEndpoint" "body"
```

The `rest` function can call arbitrary endpoints of Gandi's LiveDNS REST API. If the verb is not `GET`, the function expects a third parameter to use as the body of the `POST` or `PUT` request. Valid API endpoints can be found in the [LiveDNS API documentation](https://doc.livedns.gandi.net/). Your Gandi API key from the command line or the `~/.gandiapi` file is automatically included as an HTTP header.
