# Customation customer downloads

This repository distributes signed installers and the VS Code plugin. It contains
no vendor runtime source code. Customers do not build Docker images or clone the
vendor development repository. Setup downloads the prebuilt runtime and history
images automatically; **Run Distributed** compiles your RTL inside that runtime.

## Supported workspace host

Ubuntu 24.04 on an AMD64/x86-64 computer, with internet access and sudo rights for
initial setup. The installer installs Docker Engine, Compose and VS Code if needed.
For a Windows laptop or Mac, use VS Code Remote SSH to open a supported Ubuntu
workspace host after installing the bundle there. A native Windows/macOS installer
is not included. Docker and simulation execute on the Ubuntu host.

## Install a published release

Download the Linux AMD64 bundle from [Releases](../../releases). Once version 0.3.0
is published, run these commands on the Ubuntu workspace host as your ordinary
user (do not run the entire setup as root):

```sh
curl -fL https://github.com/cresmanoj/customation-releases/releases/download/v0.3.0/customation-0.3.0-linux-amd64.tar.gz -o customation.tar.gz
curl -fL https://raw.githubusercontent.com/cresmanoj/customation-releases/main/vendor-public.pem -o vendor-public.pem
sha256sum vendor-public.pem
tar -xzf customation.tar.gz
mkdir -p "$HOME/rtl-project"
cd customation-0.3.0-linux-amd64
./customation-setup --public-key "$(realpath ../vendor-public.pem)" --workspace "$HOME/rtl-project"
```

Before setup, confirm the public-key fingerprint against a trusted copy supplied
by the vendor. SHA-256 of the exact `vendor-public.pem` file:

`dda0ac1cff7712483c7645aec6a31202d95ef83332834a41363c854b647f781d`

The setup script verifies the signed file checksums and release manifest, installs
the plugin, pulls the exact image digests, starts the runtime, and configures the
workspace. Customers need no Google Cloud account or registry login for these
customer images. The vendor compiler build base is never downloaded by customers.

Optional: let setup clone **your own RTL/testbench repository** into a new path:

```sh
./customation-setup --public-key /trusted/vendor-public.pem \
  --workspace "$HOME/new-rtl-project" --rtl-repository YOUR_RTL_REPOSITORY_URL
```

## Run from VS Code

1. Open and trust the configured workspace. Copy your `.v`/`.sv` files into it,
   including the top-level testbench.
2. Click **Run Distributed**. First-run setup asks for source files, the top-level
   testbench/module, run length and snapshot interval. Optionally import a
   hierarchy file to choose the distributed module partitions.
3. Use the **Customation RTL** sidebar for hierarchy diagrams, recorded signals,
   hover values, pinned moments and waveforms.
4. Snapshots are automatic (10 minutes by default). After editing RTL, click
   **Run Incremental**, select changed modules, and resume the latest compatible
   snapshot. A short run may finish before its first snapshot interval.

The signed bundle's README describes the managed RTL subset and hierarchy JSON
format. The current flow limits each run to 10,000 ticks and uses a Linux controller.
RTL, simulation and history remain on your workspace host. Only release images
are downloaded; the debugger does not send your RTL to a vendor cloud service.

## Updates

Download and run setup from the new signed bundle with the same trusted key and
workspace. UI-only releases reuse existing image digests. Runtime changes are
built and published once by the vendor, before customers install them. Customers
only download the resulting images; building their RTL is part of each simulation.
