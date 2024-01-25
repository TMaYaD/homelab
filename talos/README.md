Generate secrets by running

```sh
talosctl gen secrets
```

You have to do this only once. Keep your secrets file safe.

Generate config by running

```sh
talosctl gen config \
  --with-secrets secrets.yaml \
  --config-patch @network.patch.yaml \
  --config-patch @rpi.patch.yaml \
  --config-patch @local-storage.patch.yaml \
  talos https://10.10.10.6:6443
```

Apply the generated config to the machine

```sh
talosctl apply-config -f controlplane.yaml -n <IP address>
```
