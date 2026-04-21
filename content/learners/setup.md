+++
title = "Setup"
weight = 10
+++

Snakemake can be installed in several ways. In this lesson, we use
[Pixi](https://pixi.sh/latest/) because it is fast and keeps the environment
setup simple.

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

{{< callout type="note" title="Later episodes use Apptainer" >}}
The first episodes only need Snakemake itself. The later episode on
containers also needs Apptainer, which is usually easiest on a Linux system
where Apptainer is already available, such as LxPlus or a Tier-3 cluster.

If you are working locally on macOS or Windows, you can still follow the early
episodes, but you may want to switch to such a system for the container
material.
{{< /callout >}}

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
