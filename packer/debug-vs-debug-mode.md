# Packer - Debug log vs debug mode

When it comes to troubleshooting [HashiCorp's Packer](https://www.packer.io/) builds, Packer offers 2 ways:

More [detailed logging](https://developer.hashicorp.com/packer/docs/debugging#debugging-packer) that's activated by setting the `PACKER_LOG` environment variable to a value that's not `""` nor `0` -> `PACKER_LOG=1` and

Running Packer build in [debug mode](https://developer.hashicorp.com/packer/docs/debugging#debugging-packer-builds) with `packer build -debug`

- The good: this is ideal for remote builds with cloud providers like Amazon Web Services AMIs
- The bad: disables parallelization (but) and enables debug mode (prints password for Admin).