# SharpPageGrab

A .NET utility to make web requests, similar to curl. It can be run from a PowerShell session, command prompt, or in-memory using the execute-assembly command of C2 frameworks such as Cobalt Strike or Mythic's Apollo.

Some potential use cases are:

- Checking external Internet or internal website connectivity without needing to set up a SOCKS proxy
- Getting or posting data to a REST API
- Checking SSL/TLS certificate information

It was originally released as part of [SharpUtils](https://github.com/breakid/SharpUtils), but this version was split off because the SSL/TLS checking code requires .NET Framework 4.6 to function properly.

---

## Usage

You can specify:

- The HTTP method using `-m <method>`
- A proxy server/port using `-p http(s)://<proxy>:<proxy_port>`
- POST request data, if applicable, using `-d <URL_encoded_data>`
- An arbitrary number of custom headers using `-h <name> <value>`
    - Headers must be specified in space-separated key-value pairs
    - Headers are case sensitive and will be sent exactly the way they are specified on the command-line
    - `Date`, `Host`, and `If-Modified-Since` are set automatically by .NET and cannot be overridden
    - If no `User-Agent` is specified, the program will attempt to construct a user-agent with the Edge version read from the registry
        - This is not perfect since the current version of Edge will not match the Chrome version, but it's probably better than sending no user-agent

The `-c` flag displays info about the SSL / TLS certificate. The `-v` flag causes the HTML contents of the page to be printed; if omitted, only the response headers will be shown.

```powershell
pagegrab.exe [-p http(s)://<proxy>:<proxy_port>] [-m <method>] [-d <URL encoded POST data>] [-h <header_1_name> <header_1_value> [-h <header_2_name> <header_2_value>]] [-c] [-v] <URL> [/?]
    -c  Display the SSL / TLS certificate
    -v  Print the HTML contents of the response
```

---

## Examples

```text
pagegrab.exe https://google.com -h User-Agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36 Edg/89.0.774.75"

GET https://example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36 Edg/89.0.774.75
```

```text
pagegrab.exe https://example.com -h Referer https://www.google.com -h DNT 1 -h Cache-Control no-cache
    
GET https://example.com
Proxy: None
Referer: https://www.google.com
DNT: 1
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36 Edg/92.0.902.67
```

```text
.\SharpPageGrab.exe -c https://microsoft.com
REQUEST
---------------
GET https://microsoft.com/
Proxy: None
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.1774.42 Safari/537.36 Edg/113.0.1774.42




RESPONSE
---------------
Status: OK
Connection: keep-alive
Accept-Ranges: bytes
Content-Length: 1020
Content-Type: text/html
Date: Sat, 13 May 2023 23:31:40 GMT
ETag: "6082151bd56ea922e1357f5896a90d0a:1425454794"
Last-Modified: Wed, 04 Mar 2015 07:39:54 GMT
Server: AkamaiNetStorage


SSL / TLS Certificate
---------------------
  Subject:           CN=www.microsoft.com, O=Microsoft Corporation, L=Redmond, S=WA, C=US
  Issuer:            CN=Microsoft Azure TLS Issuing CA 06, O=Microsoft Corporation, C=US
  Valid From:        10/4/2022 19:23:11 PM
  Valid To:          9/29/2023 19:23:11 PM
  Serial Number:     330059F8B6DA8689706FFA1BD900000059F8B6
  SHA-1 Fingerprint: 2D6E2AE5B36F22076A197D50009DEE66396AA99C
  Public Key:        3082010A0282010100B75F23136D6F9BFA84D14648ABA448EC1FF2621760ECCBC4A3286B824F949208EEAA7C36ADCDB505CEF327ED3145032384590863A79BE902EDF3653BBD8DE7B3B975C9C75FB6132D7956764957743BF33C0927FDF24DB8977F386A616AF7D4D9B76E81732995991990C7818EDEA40758C2539D2D09384111E6DDA24D38958281CD69AD277F5E03FEABDB183A7018551BFDB46B64CE74D7E90692F368D68C31FCCC78B6F7D2BC610A64D5AA29853049022B6BA66045487BC3830329BABA71D2B8C0BEAD50DBB9930161B4ECB31DEC9EE7C99885CD4C661430EADA2C76DB4C8F1262FEC25CF5752AB035A2198BB7FAED0F5A4391A713403920FB4AD1E629D406EF0203010001

Certificate PEM:
----BEGIN CERTIFICATE----
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
----END CERTIFICATE----

DONE
```