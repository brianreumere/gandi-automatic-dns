gad
===

This script is intended to be used as a cron job to maintain the accuracy of multiple A or AAAA records in a Gandi.net zonefile. External IP address discovery is done via a network interface, [OpenDNS](http://www.opendns.com), or a custom command piped to standard input. The result is compared to the value in the active version of the zonefile of each record in RECORD-NAMES.

Prerequisites
=============

If you are on Gandi's legacy DNS platform, your domain needs to be using a zone that you are allowed to edit. The default Gandi zone does not allow editing, so you must create a copy. There are instructions on Gandi's wiki to [create an editable zone](http://wiki.gandi.net/en/dns/zone/edit). You only need to perform the first two steps. There is a FAQ regarding this [here](http://wiki.gandi.net/en/dns/faq#cannot_change_zone_file).

If you are on Gandi's new v5/LiveDNS platform, you do not need to perform these steps.

Requirements
============

  * Bourne shell
  * OpenSSL or [LibreSSL](http://www.libressl.org)

If you're using a nutty OS that doesn't include the ifconfig or dig commands you have three options:
  * Install a package that provides the dig command, commonly bind-tools or dnsutils (to use IP discovery via OpenDNS)
  * Install a package that provides the ifconfig command, commonly net-tools (to use IP discovery via a network interface)
  * Use the -s flag and pipe a custom command that outputs your external IP address to gad, e.g., ```curl ipinfo.io/ip | gad -s -a APIKEY -d EXAMPLE.COM -r "RECORD-NAMES"```

Command-line usage
==================

```
$ gad [-5] [-6] [-l TTL] [-f] [-t] [-e] [-v] [-s] [-i EXT_IF] -a APIKEY -d EXAMPLE.COM -r "RECORD-NAMES"

-5: Use Gandi's new LiveDNS platform
-6: Update AAAA record(s) instead of A record(s)
-l: Set a custom TTL on records (only supported on LiveDNS)
-f: Force the creation of a new zonefile regardless of IP address or TTL discrepancy
-t: On Gandi's legacy DNS platform, if a new version of the zonefile is created, don't activate it. On LiveDNS, just print the updates that would be made if this flag wasn't used.
-e: Print debugging information
-v: Print information to stdout even if an update isn't needed
-s: Use stdin instead of OpenDNS to determine external IP address
-i: Use ifconfig instead of OpenDNS to determine external IP address

TTL: The custom TTL value (in seconds) to set on all records
EXT_IF: The name of your external network interface
APIKEY: Your API key provided by Gandi
EXAMPLE.COM: The domain name whose active zonefile will be updated
RECORD-NAMES: A space-separated list of the name(s) of the A or AAAA record(s) to update or create
```

For Gandi's legacy platform, request an API key [here](https://www.gandi.net/admin/apixml/). For the new LiveDNS platform, generate an API key by logging into the [account admin panel](https://account.gandi.net/) and clicking on Security.

Function syntax
============

```
rpc "methodName" "datatype" "value" "struct" "name" "datatype" "value"
```

This function can accept an arbitrary number of datatype/value pairs and structs and their member name/datatype/value tuples. structs _must_ be last! Valid method names can be found in the [Gandi API documentation](http://doc.rpc.gandi.net/index.html). Note that the APIKEY value from the command line is automatically included as the first parameter.

```
rest "verb" "apiEndpoint" "body"
```

This function can call arbitrary endpoints of Gandi's LiveDNS REST API. If the verb is not `GET`, the function expects a third parameter to use as the body of the `POST` or `PUT` request. Valid API endpoints can be found in the [LiveDNS API documentation](https://doc.livedns.gandi.net/). The APIKEY value from the command line is automatically included as an HTTP header.
