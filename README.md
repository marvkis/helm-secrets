# secrets-helm
Just a helm chart to prepare secrets with given or auto-generated secrets

## TL;DR

Add the Helm repo
```bash

helm repo add marvkis-charts https://marvkis.github.io/charts
helm repo update

```

Create a values.yaml
```yaml
secrets:
  my-secret-1:
    values:
      username:
        mode: static
        value: admin
      password:
        mode: ascii
        length: 32
  my-secret-2:
    namespace: default
    keep: false
    labels:
      hello: world
    annotations:
      planet: earth
    values:
      username:
        mode: static
        value: user
      password:
        mode: alnum
        regenerate: true
        length: 16
```

Install using the `values.yaml`
```bash
helm install --values values.yaml secrets marvkis-charts/secrets
```

To show the generated passwords you can dump the secrets using this kubectl command:

```bash
kubectl get secret my-secret-1 -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```


## Uninstalling
```bash

helm delete secrets
# secrets are not removed automatically, so they have to be removed manually
kubectl delete secrets my-secret-1

```

## Parameters

| Parameter                      | Description                                     | Default |
| ------------------------- | ----------------------------------------------- | ----- |
| `secrets` | list of secrets to create. The key is the name of the secret  |`{}` |
| `secrets.<secret>.namespace` | Can be specified if the secrets are placed into separate namespaces |`""` |
| `secrets.<secret>.labels` | Additional labels for this secret |`{}` |
| `secrets.<secret>.annotations` | Additional annotations for this secret |`{}` |
| `secrets.<secret>.keep` | Keep this secret file after helm uninstall. Set to `false` if you don't care about it. |`true` |
| `secrets.<secret>.values` | list of entries for this secret. The key is the key within the secret |`{}` |
| `secrets.<secret>.values.<key>.mode` | mode for the value. See [Value modes](#value-modes) |`""` |
| `secrets.<secret>.values.<key>.value` | the value to use when `mode`=`static` |`""` |
| `secrets.<secret>.values.<key>.length` | the length of the generated value | `16` |
| `secrets.<secret>.values.<key>.regeneratoe` | If set to `true` the secret for this key will be regenerated during each helm update/install | `false` |


### Value modes
| Mode                      | Description                                     |
| ------------------------- | ----------------------------------------------- |
| `static` | A static value taken from `value` |
| `ascii` | A random value using **all printable ASCII characters**  |
| `alnum` | A random value using **0-9a-zA-Z** |
| `alpha` | A random value using **a-zA-Z** |
| `num` | A random value using **0-9** |

