# Azure CLI Mixin for Porter

This is a mixin for Porter that provides the Azure (az) CLI.

[![Build Status](https://dev.azure.com/deislabs/porter/_apis/build/status/porter-az?branchName=main)](https://dev.azure.com/deislabs/porter/_build/latest?definitionId=20&branchName=main)

## Mixin Configuration

When you declare the mixin, you can also configure additional extensions to install

**Use the vanilla az CLI**
```yaml
mixins:
- az
```

**Install additional extensions**

```yaml
mixins:
- az:
    extensions:
    - EXTENSION_NAME
```

## Mixin Syntax

See the [az CLI Command Reference](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest) for the supported commands.

```yaml
az:
  description: "Description of the command"
  arguments:
  - arg1
  - arg2
  flags:
    FLAGNAME: FLAGVALUE
    REPEATED_FLAG:
    - FLAGVALUE1
    - FLAGVALUE2
  suppress-output: false
  outputs:
    - name: NAME
      jsonPath: JSONPATH
    - name: NAME
      path: SOURCE_FILEPATH
```

### Suppress Output

The `suppress-output` field controls whether output from the mixin should be
prevented from printing to the console. By default this value is false, using
Porter's default behavior of hiding known sensitive values. When 
`suppress-output: true` all output from the mixin (stderr and stdout) are hidden.

Step outputs (below) are still collected when output is suppressed. This allows
you to prevent sensitive data from being exposed while still collecting it from
a command and using it in your bundle.

### Outputs

The mixin supports `jsonpath` and `path` outputs.


#### JSON Path

The `jsonPath` output treats stdout like a json document and applies the expression, saving the result to the output.

```yaml
outputs:
- name: NAME
  jsonPath: JSONPATH
```

For example, if the `jsonPath` expression was `$[*].id` and the command sent the following to stdout: 

```json
[
  {
    "id": "1085517466897181794",
    "name": "my-vm"
  }
]
```

Then then output would have the following contents:

```json
["1085517466897181794"]
```

#### File Paths

The `path` output saves the content of the specified file path to an output.

```yaml
outputs:
- name: kubeconfig
  path: /root/.kube/config
```

---

## Examples

### Install the Azure IoT Extension

```yaml
mixins:
- az:
    extensions:
    - azure-cli-iot-ext
```

### Authenticate

```yaml
az:
  description: "Azure CLI login"
  arguments:
    - login
  flags:
    service-principal:
    username: "{{ bundle.credentials.AZURE_SP_CLIENT_ID}}"
    password: "{{ bundle.credentials.AZURE_SP_PASSWORD}}"
    tenant: "{{ bundle.credentials.AZURE_TENANT}}"
```

### Provision a VM

```yaml
az:
  description: "Create VM"
  arguments:
    - vm
    - create
  flags:
    resource-group: porterci
    name: myVM
    image: UbuntuLTS
```

### Delete a VM

```yaml
az:
  description: "Delete VM"
  arguments:
    - vm
    - delete
  flags:
    resource-group: porterci
    name: myVM
```