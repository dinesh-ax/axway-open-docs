{
"title": "Convert XML configurations to YAML format",
"linkTitle": "Convert XML to YAML",
"weight":"90",
"date": "2022-08-22",
"description": "Convert API Gateway federated configurations from XML to a YAML human-friendly format."
}

This section explains how to convert an existing XML federated configuration to YAML. For more information on how to create an XML federated configuration in Policy Studio, see  [Create a Policy Studio project](/docs/apim_policydev/apigw_poldev/gs_project/#new-project-from-a-template-configuration).

{{< alert title="Note">}}
Ensure that you have the most recent Policy Studio installation and upgraded your XML federated configuration to the latest version before converting it to YAML.
{{< /alert >}}

Before you start converting your configuration, observe the following:

* If your project is called, for example, `test`, then its XML files are stored at `~/apiprojects/test`, and the federated URL is `federated:file:/home/user/apiprojects/test/configs.xml`.
* If the XML federated configuration is encrypted with a passphrase, you do not need to decrypt it before or after the conversion to YAML. The converted YAML configuration remains encrypted as all values are copied from one configuration to the other.

Follow these steps to convert your XML federated configuration to YAML:

1. Open a terminal and change directory to the API Gateway installation directory `apigateway/posix/bin`.
2. Convert the XML to YAML:

    ```
    ./yamles fed2yaml -s /home/user/apiprojects/test -o /path/to/yamles
    ```

3. Validate the converted YAML configuration:

    ```
    ./yamles validate -s /path/to/yamles
    ```

For deployment to an API Gateway runtime, see [Packaging and deployment](/docs/apim_yamles/yamles_packaging_deployment/).

For troubleshooting any errors, and more details on options and commands, see [CLI Tool](/docs/apim_yamles/apim_yamles_cli).

## Edit a policy using YAML CLI

Although YAML configuration is now supported within Policy Studio, it is possible to edit a project using an IDE with YAML syntax checking and the [YAML CLI](/docs/apim_yamles/apim_yamles_cli) tool.

Building a new complex policy without an existing YAML format example might be challenging. To simplify this level of development, you can use the ES Explorer tool or Policy Studio as follows:

1. Create the structure of the policy.
2. Export the policy to a configuration fragment.
3. Convert the XML configuration fragment to YAML using the `yamles frag2yaml` command.
4. Import the YAML configuration fragment into your main YAML project using the `yamles import` command.

Repeat the export, `frag2yaml`, and `import` steps as many times as required to import all the required configuration into your main YAML configuration.

We recommend you to develop a new complex policy in an XML project that contains all the configuration you previously converted to YAML, so that you can reference other existing configuration. You must ensure that all referenced entities that need to be resolvable can be resolved eventually.

## Convert the XML configuration fragment to YAML

Certificates and private keys that exist in the XML federated configuration, which then get converted to YAML format, are formatted as-is, without header (starting line `-----BEGIN...`) and footer (ending line `-----END...`).

The related certificate and private key files in the YAML configuration will be formatted as follows:

File: `Axway-converted.yaml`

```yaml
---
type: Certificate
fields:
  dname: Axway-converted
  key: '{{file "Axway-converted-key.pem"}}'
  content: '{{file "Axway-converted-cert.pem"}}'
```

File: `Axway-converted-key.pem`

```
---
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDLd+Zv/rC6Wf/FAGc6YM8APIZD
MgEtJjOWKJiOajzI22GFsUcU3XYfV8oayI12j67IMVaD1RU+o+u6D+Y7tPj60/kvqORAjpnad7OS
0zs59KMDvfXghM3EkN8/mDxb11yK19VMJvk2ZMcpRn/6/gCFbpa6Nj8OpF7oLCRhbcue1R+JiSLF
R9zbJzxfkgkQ9Q256+ZMlOgYdrqpytTltYPOrKD3vKR9DiOTc/KC5kCZFmkgDJz5AQTpTtvZCXJW
aro+PbAqbyc//ABTTrhTIBkyB+aFM49hg/Ej90KitvxI/Nu4lzT2XiDd9wqiTzElYM+Q0UMk0BFd
vnckh1sLYHe3AgMBAAECggEABK9VEf0WSqQp3Hpe5hw2h/XczY1IM6buhyWWJalSjvlmLHLhhRx4
TM5zq9w0TaePSbLBIDX20ENr+RPGpFdNaFEbKrrDzqy55CrfaqEMexAj9MEZ+Tp1lnITgd5afW7f
BZ9knOVE1bjKUSv7ZGcW0fuy4sS+/PJR8RybFdc2WgjZt0kw8L9fUsmq/M4ye00DVOoI/eTGjfky
ooVtUAJJXYI1FMvUuyC43qJuOMpfme3xY79AOQpXBvTamG/xpr7/MX5Q4yxu5GEo7eLk91xwSKFG
OYgC33SLgxOIRecLxgxZp72g4GZqGLzcjmTDxpMgS32RzbRtqDNCp5Twjvw0aQKBgQDmzo6WAMa9
jCbEtXsqCQ4CkCYbMJp3d9Jxzk6R2kjwMRQLDL+P+YkIMd/tntrd3DS2vbdbzlGxFkDROhpV+T+T
iYlqFkY60cUaeGBWyWTWkKksrLfsJz4M5/seflYX4L9H5vjDChybzpaViTUHof1xB6f44fheeW82
sV7v+UQQ9QKBgQDhrWwtGJjccJgAfZwqnBd0snKDBUjpSlUgBQnyYzNXbM9IN9q4vaHymt5Zgg5f
do4Bfjdmx4O3W6RtIysmw92hyad8SIbl2JNN15FwQQ2ZpgaIgsUyVQGywtkiXzFGnir3YyEjtb/S
5y1X4Au2mnof1K2Q9UYUKeEsoLFNDj/KewKBgQCddz8AR/dPSlcIzWgB/bt5NC9LTZWU/EKvMjmY
eHxaoqEyCLiI2Y4L8Tr9OuvHgXzVUAnQsKo7TxtZo3JkRXCCj8sYfancZ1E6BfZ0P2J0oK1KtWul
ygAjfgFthHPoRoU54PLG3hc2lXNXAg0T8AihHMAUpNZ2XhLqFYjX4A/4IQKBgQCldRF3qq4ACKjY
yz8g2lo0G9TrDIfdSrtIg4k8ZdCxizwZ1aGNirLefP8CHuFMyk3o+FHEEAkY+J5/yaYMgNPQl1kt
PLtybqvpCWA/LeK7wMbPdRkBAuQA3Ox3T9V/0dzsjYgxd0JRbV6IK+JKmc1p4vLx8XHUvLOzlYkI
VqccWwKBgByv6gva+WWT5H7CZCicURkLdkRx0RGJjZKW5m7uwWPLAkb5zVozsCSGhNZDX3xuPzqx
z+m92RPtpagPgowuDomFodteFZOsSjCkvbiwx7JGEsvwEraMwxYGPg3sEpVa+bS/4Ptpj8llGMpN
NWjvpIPxQaBeo3FHRDPFjvc7qjUP
```

File: `Axway-converted-cert.pem`

```
---
MIICrTCCAZUCBgE9RY7XxjANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDEw9TYW1wbGVzIFRlc3Qg
Q0EwHhcNMTMwMzA3MTU1MzAwWhcNMzcwMTEwMTA1NjAwWjAaMRgwFgYDVQQDEw9TYW1wbGVzIFRl
c3QgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDLd+Zv/rC6Wf/FAGc6YM8APIZD
MgEtJjOWKJiOajzI22GFsUcU3XYfV8oayI12j67IMVaD1RU+o+u6D+Y7tPj60/kvqORAjpnad7OS
0zs59KMDvfXghM3EkN8/mDxb11yK19VMJvk2ZMcpRn/6/gCFbpa6Nj8OpF7oLCRhbcue1R+JiSLF
R9zbJzxfkgkQ9Q256+ZMlOgYdrqpytTltYPOrKD3vKR9DiOTc/KC5kCZFmkgDJz5AQTpTtvZCXJW
aro+PbAqbyc//ABTTrhTIBkyB+aFM49hg/Ej90KitvxI/Nu4lzT2XiDd9wqiTzElYM+Q0UMk0BFd
vnckh1sLYHe3AgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAECtUaJVPApyeJrxqb3j49QbtL3rhusQ
ae70epkkU0XFu+8NuYoz4pVs5Mj9g0HPXXvRvgJAQzCzQ34XVFZGB18IjOkoyu0RodTadGi7fAH5
0B/SEteGRhYV1JzE+IKxG3n6nTrrHgINL8+QL4eHiozEKfJ/vWqw1j2Zc/47vcalGYYqTJyrSmJy
pX6tVTYRbArvwaU+LzYLjEJkWgfZc09eO8JiA1ZwE4ud5o9Tpbj5IdUQkoLyPTxtU0a6A2PMAs4j
MTnD3Mh2YRB4gGCAPDN+Xl999ls2KCoVdLjXIhOD2YwWXGmv3vaAUyc32wwcro69/wXvj41tFIez
w98SGM8=
```

## Convert Policy Studio environmentalization for XML configurations to YAML

You can environmentalize XML federated configurations in Policy Studio by clicking **Preferences > Environmentalization > Allow environmentalization**, and using the `{{env "XYZ"}}` syntax.

Observe the following when XML federated configurations that contain fields environmentalized via Policy Studio are converted to YAML format:

* A `values.yaml` is created.
* Each environmentalized field with a value yields an entry:
    * In `values.yaml`, with the environmentalized values.
    * In `values-original.yaml`, with the value of the field before being environmentalized in Policy Studio.
* A placeholder replaces the value in entity YAML file. The naming follows the YamlPK logic with sanitization that is compliant with the environmentalization syntax.

The following is an example of an entity with an environmentalized URL, username, and password:

**Entity in XML federated configuration**:

* Type: `LdapDirectory` (LDAP Connection)
* Name: `api-env LDAP`
* Environmentalized fields: `url`, `userName`, `password`
* Contained in `LDAP Connections`

**YAML configuration after conversion**:

```yaml
---
type: LdapDirectory
fields:
  name: api-env LDAP
  url: '{{LDAP_Connection.api_env_LDAP.url}}'
  userName: '{{LDAP_Connection.api_env_LDAP.userName}}'
  password: '{{LDAP_Connection.api_env_LDAP.password}}'
```

**Content of values.yaml**:

```yaml
LDAP_Connection:
  api_env_LDAP:
    url: ldap://api-env:389
    userName: cn=Administrator,dc=demo.axway,dc=com
    password: KLJH95dHS7djshkjas54sa45s4d==
```

The following is an example of an entity with an environmentalized certificate:

**Entity in XML federated configuration**:

* Type: `ConnectToURLFilter`
* Name: `name: Connect to URL`
* Environmentalized Fields: `sslUsers`
* Contained in `Policies/YAML Demo/Connect with SSL`

**YAML configuration after conversion**:

```yaml
---
type: FilterCircuit
fields:
  name: Connect with SSL
  start: ./Connect to URL
  description: ""
children:
- type: ConnectToURLFilter
  fields:
    name: Connect to URL
    sslUsers: '{{YAML_Demo.Connect_with_SSL.Connect_to_URL.sslUsers}}'
    url: https://localhost:5555/cert-verifier
    name: Connect to URL
# ...
```

**Content of values.yaml**:

```yaml
---
YAML_Demo:
  Connect_with_SSL:
    Connect_to_URL:
      sslUsers: /Environment Configuration/Certificate Store/O=DEV,CN=localhost
```