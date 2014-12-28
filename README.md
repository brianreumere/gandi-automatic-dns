gad
===

This script is intended to be used as a cron job to maintain the accuracy of multiple A or AAAA records in Gandi.net zonefiles. External IP address discovery is done via a network interface (works on OpenBSD and Linux with net-tools 1.60-23, untested on anything else) or [OpenDNS] [1]. This is compared to the value in the active version of the zonefile of each record in RECORD-NAMES. Using the rpc() function to update different types of DNS records or call other methods of Gandi's XML-RPC API should be fairly trivial.

Prerequisites
=============

Your domain needs to be using a zone that you are allowed to edit. The default Gandi zone does not allow editing, so you must create a copy. There are instructions on Gandi's wiki to [create an editable zone] [2]. You only need to perform the first two steps. There is a FAQ regarding this [here] [3].

Requirements
============

  * Bourne shell
  * OpenSSL

Command line usage
==================

```
$ gad [-6] [-f] [-t] [-v] [-i EXT_IF] -a APIKEY -d EXAMPLE.COM -r "RECORD-NAMES"

-6: Update AAAA record(s) instead of A record(s)
-f: Force the creation of a new zonefile regardless of IP address discrepancy
-t: If a new version of the zonefile is created, do not activate it
-v: Print information to stdout even if a new zonefile isn't needed
-i: Use ifconfig instead of OpenDNS to determine external IP address

EXT_IF: The name of your external network interface
APIKEY: Your API key provided by Gandi
EXAMPLE.COM: The domain name whose active zonefile will be updated
RECORD-NAMES: A space-separated list of the name(s) of the A or AAAA record(s) to update or create
```

Request an API key from Gandi [here] [4].

rpc() syntax
============

```
rpc "methodName" "datatype" "value" "struct" "name" "datatype" "value"
```

This function can accept an arbitrary number of datatype/value pairs and structs and their member name/datatype/value tuples. structs _must_ be last! Valid method names can be found in the [Gandi API documentation] [5]. Note that the APIKEY value from the command line is automatically included as the first parameter.

  [1]: http://www.opendns.com
  [2]: http://wiki.gandi.net/en/dns/zone/edit
  [3]: http://wiki.gandi.net/en/dns/faq#cannot_change_zone_file
  [4]: https://www.gandi.net/admin/apixml/
  [5]: http://doc.rpc.gandi.net/index.html
