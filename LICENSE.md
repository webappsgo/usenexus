MIT License

Copyright (c) 2026 webappsgo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Embedded Licenses

This software includes the following third-party libraries:

| Library | Version | License | Copyright |
|---|---:|---|---|
| github.com/oschwald/maxminddb-golang/v2 | 2.9.1 | ISC | Copyright (c) 2015, Gregory J. Oschwald |

## Third-Party Data Sources

GoSpool downloads the following GeoIP databases from [sapics/ip-location-db](https://github.com/sapics/ip-location-db):

| Database | Underlying source | License |
|---|---|---|
| `iptoasn-asn.mmdb` | iptoasn.com (RouteViews / RIR data) | PDDL |
| `iptoasn-country.mmdb` | iptoasn.com (RIR data) | PDDL |
| `dbip-city-ipv4.mmdb`, `dbip-city-ipv6.mmdb` | DB-IP | CC BY 4.0 |

The City databases require this attribution:

<a href="https://db-ip.com/">IP Geolocation by DB-IP</a>
