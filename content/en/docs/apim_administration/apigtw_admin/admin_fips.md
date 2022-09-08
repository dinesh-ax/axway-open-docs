{
"title": "Run API Gateway in FIPS mode",
"linkTitle": "Run API Gateway in FIPS mode",
"weight":"85",
"date": "2019-10-14",
"description": "Run an API Gateway instance or a Policy Studio client in FIPS mode."
}

API Gateway supports Federal Information Processing Standards (FIPS). When running an API Gateway instance or a Policy Studio client in FIPS mode, the following FIPS-certified cryptographic modules are enabled and invoked for all FIPS-compliant cryptographic algorithms:

| Cryptographic Module                                          | FIPS 140-2 Certificate Number |
|---------------------------------------------------------------|-------------------------------|
| Entrust Authority Security Toolkit for the Java Platform v8.0 | 1839                          |
| OpenSSL FIPS Object Module                                    | 1747                          |

{{< alert title="Note" color="primary" >}}
Running API Gateway in FIPS mode is a separately licensed option that must be specifically ordered. For more details, contact your Axway sales representative.
{{< /alert >}}

## Enable FIPS mode for an API Gateway

You can use the `togglefips` script to enable or disable FIPS mode for a gateway instance.

* You can run this script only with a FIPS-enabled API Gateway license.
* You must restart any Node Manager or API Gateway instances after running the `togglefips` script.
* To enable FIPS in a multi-node domain you must run `togglefips --enable` on all nodes in the domain, and all API Gateway processes must be restarted.

Run the following commands:

```
> cd apigateway/posix/bin
> ./togglefips --enable | -e
> ./togglefips --disable | -d
```

## Enable FIPS mode for Policy Studio

You can also enable FIPS mode for a Policy Studio client application only. To enable or disable FIPS mode in Policy Studio, perform the following steps:

1. Select **Preferences > FIPS Mode**.
2. Select **Enable FIPS Mode in Axway Policy Studio**.
3. Restart Policy Studio with the `clean` option as follows:

```
policystudio -clean
```

You can use the same instructions to enable FIPS in Configuration Studio.

## Restrictions when running in FIPS mode

When running in FIPS mode, certain gateway features are not enabled because they depend on non-FIPS compliant algorithms.

The following features cannot be run when the gateway is running in FIPS mode:

* HTTP digest authentication filter
* Kerberos authentication where MD5, DES, and other non-FIPS compliant algorithms are used

For a complete list of non-FIPS compliant algorithms and cipher suites configured in all crypto-related filters and interfaces in Policy Studio, select **Tools > Check Security Constraints > FIPS**, and view the output on the pane on the right.

### Engines are not supported in FIPS mode

Since OpenSSL 3.0 has moved from Engines to Providers, `OPENSSL_ENGINES` are no longer supported in FIPS mode. For FIPS compliance, you must use Provider APIs.

## Further information

For more details on FIPS, see [Federal Information Processing Standards Publications (FIPS PUBS)](http://www.nist.gov/itl/fips.cfm).
