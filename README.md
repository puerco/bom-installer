# bom-installer

### GitHub action to install `bom`, the Kubernetes SBOM tool

This GitHub action installs `bom`,
[the Kubernetes SBOM tool](https://github.com/kubernetes-sigs/bom)
that allows users to create a SPDX bill of materials using the same
code that powers the Kubernetes SBOM generation produced with every
release.

## Installation Example

The following example will install `bom` in your GitHub Actions 
runner, generate an SBOM describing the code in your repository and
will output it to STDOUT. You should be able to see the SBOM in 
the Actions console.

Add the following code to `.github/workflows/sbom.yaml`, then
go to the Action tab in your proyect and trigger it:

```yaml
name: sbom
on:
    workflow_dispatch:
        inputs:
            bomVersion:
                description: 'Version of bom to use for SBOM generation'
                required: true
                default: 'v0.2.1'
                type: choice
                options:
                    - v0.3.0
                    - v0.2.2
                    - v0.2.1
jobs:
    generate-sbom:
        # v0.2.1 linux binaries are compatible with the ubuntu runners. 
        runs-on: macos-latest
        permissions:
            # This is a demo, so no permissions beyond read are required
            contents: read       
        steps:
            # Call the bom-installer action (make sure to pin it to the last commit)
            - name: Install Kubernetes bom
              uses: puerco/bom-installer@ee0001c9a661b75c7c72426f50811691e7901590
              with:
                bom-release: ${{ github.event.inputs.bomVersion }}
            - name: Checkout
              uses: actions/checkout@v2
            # Generate an sbom from the current directory. You can pass any
            # parameters here to bom (see bom generate --help for mor info).
            #
            # For more control and better SBOM composition options, we recommend
            # using a YAML configuration file in your repository
            - run: bom generate .
```

## Configuring the SBOM structure

The installer action allows full access to the `bom` utility command line flags
but to compose better and more complex SBOMs, we recommend adding a `.sbom.yaml`
configuration file to your repository and pass it to `bom` via the `--config` flag.

A brief example of a configuration file:

```yaml
---
namespace: https://example.com/
license: Apache-2.0
name: myproject
creator:
    person: Author Name (email@example.com)
artifacts:
    - type: directory
      source: .
      gomodules: true
    - type: file
      source: bin/linux-binary
    - type: file
      source: bin/mac-binary
    - type: file
      source: bin/windows-binary.exe
    - type: image
      source: ghcr.io/myorg/myrepo/myimage:tag
      license: Apache-2.0

```

This example would produce an SBOM with the following structure (sample):

```
 📂 SPDX Document SBOM-SPDX-ed8052cf-d9ac-44f3-862e-a80bad66bb9b
  │ 
  │ 📦 DESCRIBES 2 Packages
  │ 
  ├ directoryName
  │  └ 🔗 130 Relationships
  │ 
  ├ ghcr.io/myorg/myrepo/myimage:tag
  │  └ 🔗 2 Relationships
  │ 
  │ 📄 DESCRIBES 3 Files
  │ 
  ├ SPDXRef-File-bin-linux-binary (bin/linux-binary)
  ├ SPDXRef-File-bin-windows-binary-exe (bin/windows-binary.exe)
  └ SPDXRef-File-bin-mac-binary (bin/mac-binary)
```

You can check the structure of any SPDX SBOM in tag-value format with 
bom by running `bom document outline sbom.spdx`

## Getting Help

For more information please check the documentation on the
[bom project page](https://github.com/kubernetes-sigs/bom) or reach out to the 
Release Engineering team in the 
[#release-management channel](https://app.slack.com/client/T09NY5SBT/CJH2GBF7Y)
in Kubernetes Slack. 

