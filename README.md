# invoicesec
Security test suite for electronic invoices

See also: <https://invoice.secvuln.info/>

This repository provides test cases / exploits for potential XML security
vulnerabilities in electronic invoice software.

All test cases are available in CII and UBL formats. The subdirectory [examples](
examples/) contains simple example files without exploits.

## XXE

The subdirectory [xxe](xxe/) contains various test cases for a simple XXE attack.

All test cases include a file from the local filesystem through an entity and add it to
both the seller's and buyer's names in the invoice. Any software that reads and displays
invoices can be tested.

E.g., the file [ciixxepasswd.xml](xxe/ciixxepasswd.xml) tries to exfiltrate the content
of `/etc/passwd` (usually available on all Linux and other Unix-style systems). The file
[ciixxesystemini.xml](xxe/ciixxesystemini.xml) tries to exfiltrate
`C:\Windows\system.ini`, usually available on Windows systems.

XML also supports a feature called XInclude, which allows including files through a
different mechanism. Test cases for XInclude are also provided. However, such
vulnerabilities are less common than entity-based XXE flaws, as XInclude is rarely
enabled by default.

## Blind XXE

In cases where the user does not see any output, a blind XXE may be able to exfiltrate
files. Test cases are provided in the [bxxe](bxxe/) directory.

Blind XXE attacks use so-called parameter entities to construct a URL with the content
of a file appended. They require an online and an offline component. To test these, the
following steps need to be performed:

* Adapt the hostname in all files from `bxxe.example.com` to a host you control. The
  script `makebxxe` automates this and creates a copy of the test cases with an adapted
  hostname.

* You need to run an HTTP server with logging enabled on the host and upload the content
  of the `online` directory to the root of the HTTP server.

In case of a successful attack, you will see requests like this in the web server log:

```
"GET /bxxehostname.dtd HTTP/1.1" 200 427 "-" "Java/21.0.8"
"GET /ex.txt?e=[content] HTTP/1.1" 200 282 "-" "Java/21.0.8"
```

Note that most XML parsers will not accept URLs with special characters or newlines in
them. Therefore, test cases for files that have a higher chance of being successfully
exfiltrated. Those include `/etc/hostname` (often contains a trailing slash, which some
XML parsers will accept) and `/proc/self/loginuid` (usually available on Linux systems,
contains only numbers and no newline).

Furthermore, test cases using PHP's base64 filter URLs are included. Those work when XML
files are parsed with PHP and entities are enabled. (Entities are disabled by default in
PHP's XML parsers.)

Some older versions of Java reject `http://` URLs with newlines, but allow them in
`ftp://` URLs. Therefore, test cases with `ftp://` URLs are also included. Those require
operating an FTP server with detailed logging on the exfiltration host.

# more

Currently not included are tests for denial of service issues like the Billion Laughs
Attack. Also not included are test cases in hybrid PDF/XML invoice formats (e.g.,
ZUGFeRD). Those may be added in the future.

# about

Created by [Hanno Böck](https://itsec.hboeck.de/).

The invoice examples are based on [example documents](
https://github.com/ConnectingEurope/eInvoicing-EN16931) licensed as EUPL-1.2. The
content of this repository is therefore also licensed as EUPL-1.2.
