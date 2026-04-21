+++
title = "Setup"
weight = 10
+++

Snakemake is a Python library, which is distributed through
[Bioconda](https://bioconda.github.io/recipes/snakemake/README.html).
In order to install it, you need a tool such as Conda/Mamba.
We recommend to use Pixi since we find it to be faster and easier
to use than the other tools.

{{< callout type="note" title="Detailed Installation Instructions" >}}
You can find the full installation instructions at
<https://pixi.prefix.dev/latest/installation/>.
{{< /callout >}}

If CVMFS is available, we recommend to source the `setup.sh` as
described below.

{{< tabs >}}
{{< tab name="LxPlus/CVMFS" selected=true >}}
```bash
source /cvmfs/cms-griddata.cern.ch/cat/sw/pixi/latest/setup.sh
```
{{< /tab >}}
{{< tab name="Linux/macOS" >}}
```bash
curl -fsSL https://pixi.sh/install.sh | sh
```
{{< /tab >}}
{{< tab name="Windows" >}}
```powershell
powershell -ExecutionPolicy Bypass -c "irm -useb https://pixi.sh/install.ps1 | iex"
```
{{< /tab >}}
{{< /tabs >}}

Check that `pixi` is available:

```bash
pixi version
```

## Next step

Now you're ready to [start using Snakemake]({{< relref "/episodes/getting-started" >}}).
