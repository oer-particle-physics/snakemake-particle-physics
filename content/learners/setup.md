+++
title = "Setup"
weight = 10
+++

Snakemake is a Python library, which is distributed through
[Bioconda](https://bioconda.github.io/recipes/snakemake/README.html).
In order to install it, you need a tool such as Conda/Mamba.
We recommend to use Pixi since we find it to be faster and easier
to use than the other tools.

## Installing Pixi

{{< callout type="note" title="Detailed installation instructions" >}}
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

## Installing Snakemake

We will use a dedicated directory for this tutorial:

```bash
mkdir snakemake-tutorial
cd snakemake-tutorial
pixi init
pixi workspace channel add conda-forge
pixi workspace channel add bioconda
pixi add snakemake
```

{{< callout type="note" title="Alternative installation instructions" >}}
You can find several more options to install Snakemake at
<https://snakemake.readthedocs.io/en/stable/getting_started/installation.html>.
{{< /callout >}}

To activate the environment, run:

```bash
pixi shell
```

Snakemake should then be available:

```bash
snakemake --help
```

You can also run Snakemake without activating the `shell`
(type `exit` to get out of it) directly through `pixi`:

```bash
pixi run snakemake --help
```

## Next step

Now you're ready to [start using Snakemake]({{< relref "/episodes/getting-started" >}}).
