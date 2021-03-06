# policy

*policy* is the Policy Enforcement Point for Themis project implemented as CoreDNS plugin.

## Syntax

~~~ txt
policy {
    endpoint ADDR_1, ADDR_2, ... ADDR_N
    edns0 CODE NAME [SRCTYPE DSTTYPE] [SIZE START END]
    ...
    debug_query_suffix SUFFIX
    debug_id ID
    streams COUNT
    transfer ATTR_1, ATTR_2, ... ATTR_N
    dnstap ATTR_1, ATTR_2, ... ATTR_N
    passthrough SUFFIX_1, SUFFIX_2, ... SUFFIX_N
    log
}
~~~

Option endpoint defines set of PDP addresses

Option edns0 is used for parsing edns0 options into PDP attributes, option with CODE is parsed as attribute with NAME.

Valid SRCTYPE are hex (default), bytes, ip.

Valid DSTTYPE depends on Themis PDP implementation, ATM is supported string (default), address.

Params SIZE, START, END is supported only for SRCTYPE = hex.

Set param SIZE to value > 0 enables edns0 option data size check.

Param START and END (last data byte index + 1) allow to get separate part of edns0 option data.

Option debug_query_suffix SUFFIX (should have dot at the end) enables debug query feature.

Option debug_id set string that is used for debug query response as unique id for determine what CoreDNS instance replies on the request.

Option streams set gRPC streams count for PDP connection.

Option transfer defines set of attributes (from domain validation response) that should be inserted into IP validation request.

Option dnstap defines attributes to be included in extra field of DNStap message if received from PDP.

Option passthrough defines set of domain name suffixes, domain that contains one of these is resolved without validation, each suffix should have dot at the end.

Option connection_timeout sets timeout for query validation when no PDP server are available. Negative value or "no" keyword means wait forever. This is default behavior. With zero timeout validation fails instantly if there is no PDP servers. The option works only if gRPC streams are greater than 0.

Option log enables log PDP request and response

## Example

~~~ txt
policy {
    endpoint 10.0.0.7:1234, 10.0.0.8:1234
    edns0 0xffee client_id hex string 32 0 32
    edns0 0xffee group_id hex string 32 16 32
    edns0 0xffef uid // equal edns0 0xffef uid hex string
    edns0 0xffea source_ip ip address
    edns0 0xffeb client_name bytes string
    debug_query_suffix debug.
    debug_id instance_1
    streams 100
    transfer gid uid
    dnstap pid
    passthrough mycompanyname.com. mycompanyname.org.
    log
}
~~~

In this case edns0 options with code 0xffee is splitted into two values - client_id (first 16 bytes) and group_id (last 16 bytes), option should have size 32 bytes otherwise client_id and group_id is not parsed.

Dig command example for debug query:
~~~ txt
dig @127.0.0.1 msn.com.debug txt ch
~~~
