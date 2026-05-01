# flatcar-k3s-example

This is a quickly sanitized and stripped down example of how to use Butane to create a Flatcar Linux installation, that initializes a k3s cluster on `k3s-srv-1` and joins 2 additional servers to it. The Butane files are in `flatcar/butane/` and running `make new-cluster` with `flatcar` as working directory will create the ignition files in `flatcar/ignition/`.

This config is not meant to be used as is, since it contains some Hetzner bare metal specific configuration regarding networking.

Once the cluster is up and running, [flatcar-linux-update-operator](https://github.com/flatcar/flatcar-linux-update-operator) is deployed to keep Flatcar Linux up to date.
If an update is available, the operator will cordon and drain the nodes, update them and reboot them one by one within the defined maintenance window. This way, the cluster is always running the latest Flatcar Linux version without any manual intervention.

## Disclaimer

If anyone is seriously interested in using this as a base for their own cluster and is stuck, feel free to reach out to me. I will try to help as best as I can. But please keep in mind that this is just a quick example and not meant to be used as is. It is also not meant to be a complete guide on how to set up a k3s cluster on Flatcar Linux, but rather a starting point for anyone who wants to do so.