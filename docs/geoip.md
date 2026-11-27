# GeoIP country controls

GoSpool uses the ASN, country, and city MaxMind DB files published by
[sapics/ip-location-db](https://github.com/sapics/ip-location-db). The files are
downloaded on first start and refreshed by the built-in `geoip_update`
scheduled task. Each asset is checked against its upstream SHA-256 file before
it replaces the installed copy. The databases live under the data directory's
`security/geoip` folder; they are not embedded in the binary.

GeoIP is a deny-only risk signal. It never authenticates a request or grants
access. Country blocking runs after rate limiting and before route
authentication. Requests continue through the normal authentication,
authorization, validation, and audit checks when a lookup is successful and
the country is not blocked. Missing, stale, or unreadable databases fail open;
private, loopback, link-local, unspecified, and multicast addresses are never
looked up or country-blocked. Data older than 21 days is considered stale.

The default configuration enables GeoIP but blocks no countries:

```yaml
server:
  geoip:
    enabled: true
    deny_countries: []
    allow_countries: []
    presets: {}
```

Country codes use uppercase ISO 3166-1 alpha-2 form. A nonempty `allow_countries`
list takes precedence over `deny_countries`. For example, `deny_countries:
[CN, RU]` blocks those two countries; `allow_countries: [US, CA, GB]` allows
only those three. Allowlisted IPs bypass country blocking and rate limiting,
but still require authentication and authorization.

The same accept-time filter applies to HTTP, NNTP/NNTPS, and inbound SMTP.
For NNTP and SMTP, a blocked connection is closed before the protocol greeting.
Country data can be bypassed with VPNs, proxies, or Tor and must not be used as
the sole control for a protected resource.

The country code, ASN, organization, city, state, postal code, coordinates, and
timezone are available to server-side lookups. Public API responses do not
expose a visitor's location.

City data is licensed under CC BY 4.0 and requires the visible
[IP Geolocation by DB-IP](https://db-ip.com/) attribution on `/server/about`.
The downloaded ASN and country databases use PDDL. The source and license
inventory is in the repository's [LICENSE.md](https://github.com/webappsgo/usenexus/blob/main/LICENSE.md).
