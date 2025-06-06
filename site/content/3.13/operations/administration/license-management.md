---
title: Enterprise Edition License Management
menuTitle: License Management
weight: 20
description: >-
  How to check and activate licenses for ArangoDB Enterprise Edition deployments
---
The Enterprise Edition of ArangoDB requires a license to activate the
Enterprise Edition features. How to set a license key and to retrieve
information about the current license via the JavaScript API is described below.
You can also use an [HTTP API](../../develop/http-api/administration.md#license).

If you use the ArangoDB Kubernetes Operator, check the
[kube-arangodb documentation](https://arangodb.github.io/kube-arangodb/docs/how-to/set_license.html)
for more details on how to set a license key.

## Active a license

On the first installation of any ArangoDB Enterprise Edition instance, you can
immediately use it for testing without restrictions for three hours.

In the email with the download link, you find a fully featured but
time-wise limited license that allows you to continue testing for two weeks.

You can apply this evaluation license or a proper license you bought via
_arangosh_ like so:

```js
db._setLicense("<license-string>");
```

You receive a message reporting whether the operation has been successful.
Please be careful to copy the exact string from the email and to put it in
quotes as shown above.

```json
{ "error": false, "code": 201 }
```

Your license has now been applied.

## Check the current license

At any point, you may check the current state of your license in _arangosh_:

```js
db._getLicense();
```

```json
{
  "features": {
    "expires": 1632411828
  },
  "license": "JD4E ... dnDw==",
  "version": 1,
  "status": "good",
  "hash": "..."
}
```

The `status` attribute is the executive summary of your license and
can have the following values:

- `good`: Your license is valid for more than another 1 week.
- `expiring`: Your license is about to expire shortly. Please contact
  your ArangoDB sales representative to acquire a new license or
  extend your old license.
- `read-only`: Your license has expired at which
  point the deployment will be in read-only mode. All read operations to the
  instance will keep functioning. However, no data or data definition changes
  can be made. Please contact your ArangoDB sales representative immediately.

The attribute `expires` in `features` denotes the expiry date as Unix timestamp
(in seconds since January 1st, 1970 UTC).

The `license` field holds an encrypted and base64-encoded version of the
applied license for reference and support from ArangoDB.

## Monitoring

In order to monitor the remaining validity of the license, the metric
`arangodb_license_expires` is exposed by Coordinators and DB-Servers, see the
[Metrics API](../../develop/http-api/monitoring/metrics.md).

## Managing Your License

Backups, restores, exports and imports and the license management do not
interfere with each other. In other words, the license is not backed up
and restored with any of the above mechanisms.

Make sure that you store your license in a safe place, and potentially the
email with which you received it, should you require the license key to
re-activate a deployment.
