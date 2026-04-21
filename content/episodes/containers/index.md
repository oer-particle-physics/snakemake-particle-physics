+++
title = "Containers"
weight = 30
teaching = 15
exercises = 10
questions = [
  "How can I run a workflow step in a controlled software environment?",
  "What changes when I add a `container:` directive to a rule?",
  "What do I need in order to run containerised rules?"
]
objectives = [
  "Use the `container:` directive in a rule.",
  "Run Snakemake with Apptainer using the current command-line syntax.",
  "Understand why different rules can use different software environments.",
  "Recognise which container details depend on the local site configuration."
]
keypoints = [
  "**container:** lets a rule declare the software environment it needs.",
  "**`--software-deployment-method apptainer`** tells Snakemake to execute containerised rules with Apptainer.",
  "**Per-rule containers** keep workflow logic and software requirements explicit.",
  "**Some details depend on where you run the workflow**, for example whether Apptainer is already installed or whether extra bind mounts are needed."
]
+++

In particle physics, we often need software that is awkward to install or keep
consistent across different machines. One step may need a modern Python stack,
another may need CMSSW, and a third may depend on a specific ROOT build.

Snakemake lets a rule describe not only its inputs and outputs, but also the
software environment it should run in. That makes workflows more portable and
more reproducible, because the rule can carry its environment with it.

## Why Containers Matter

Without containers, your workflow depends on whatever happens to be installed on
the machine where it runs. With containers, the workflow can say exactly which
environment a given step should use.

This is especially useful in HEP because:

- the same workflow may run on a laptop, on a shared system, or on batch nodes
- different rules may genuinely need different software stacks
- the analysis logic should stay the same even when the execution environment
  changes

## A First Containerised Rule

We can reuse the `plot.py` script from the previous episode and run the gather
step inside a Python container.

```python
DATASETS = ["DYJets", "TTbar", "Data"]
CHUNKS = ["0", "1", "2"]


rule all:
    input:
        "plots/event_counts.txt"


rule select_events:
    input:
        "input/{dataset}/{dataset}.{chunk}.txt"
    output:
        "selected/{dataset}/{dataset}.{chunk}.txt"
    shell:
        """
        mkdir -p "selected/{wildcards.dataset}"
        grep "Selected" "{input}" > "{output}" || test $? -eq 1
        """


rule make_plot:
    input:
        expand(
            "selected/{dataset}/{dataset}.{chunk}.txt",
            dataset=DATASETS,
            chunk=CHUNKS,
        )
    output:
        "plots/event_counts.txt"
    container:
        "docker://python:3.11-slim"
    script:
        "plot.py"
```

The workflow logic is unchanged: `make_plot` still gathers the selected files
and writes the summary. The new part is the `container:` directive, which tells
Snakemake which image to use for that rule.

## Running the Workflow with Apptainer

To make Snakemake execute containerised rules, use:

```bash
pixi run snakemake --cores 4 --sdm apptainer
```

{{< callout type="warning" title="What if you omit `--sdm apptainer`?" >}}
If you run Snakemake without enabling Apptainer, the `container:` directive is
ignored and the rule runs in your ordinary host environment instead.

That means the workflow may still appear to work if the required software is
already installed on your machine, but you are no longer actually testing the
containerised version of the rule.
{{< /callout >}}

{{< callout type="note" title="Practical notes" >}}
If your system defines `TMPDIR`, it is a good idea to keep the Apptainer cache
there rather than in your home directory:

```bash
export APPTAINER_CACHEDIR="${TMPDIR}/.apptainer-cache"
mkdir -p "${APPTAINER_CACHEDIR}"
```

If `TMPDIR` is not defined, use a user-specific directory under `/tmp`
instead:

```bash
export APPTAINER_CACHEDIR="/tmp/${USER}/.apptainer-cache"
mkdir -p "${APPTAINER_CACHEDIR}"
```

Older tutorials may use `--use-apptainer` or `--use-singularity`. In this
lesson, we use `--software-deployment-method apptainer` or the short form
`--sdm apptainer`, because that matches the current Snakemake documentation.

If you want, you can also store the command and cache setting in the generated
`pixi.toml`:

```toml
[tasks]
snakemake-apptainer = { cmd = "snakemake --cores 4 --sdm apptainer", env = { APPTAINER_CACHEDIR = "$TMPDIR/.apptainer-cache" } }
```

You can then run:

```bash
pixi run snakemake-apptainer
```

For more on Pixi tasks, see the
[Pixi advanced tasks documentation](https://pixi.prefix.dev/latest/workspace/advanced_tasks/)
and the
[Pixi environment variable documentation](https://pixi.prefix.dev/latest/reference/environment_variables/).
{{< /callout >}}

## What Happens Behind the Scenes?

When Snakemake sees a rule with a `container:` directive and you run with
Apptainer enabled, it will:

1. resolve the requested image
1. pull it if needed and cache it locally
1. execute the job inside that container
1. make the workflow files available to the job

From the rule author's point of view, this is the important part: the rule
still declares `input`, `output`, and how to run the step. The container just
defines the software environment for that step.

## Why Per-Rule Containers Are Useful

Different parts of a workflow may need different environments. Snakemake allows
that naturally:

```python
rule old_software_step:
    output:
        "intermediate.root"
    container:
        "docker://my-old-root-image:latest"
    shell:
        "run_old_code > {output}"


rule modern_python_step:
    input:
        "intermediate.root"
    output:
        "plots/final_summary.txt"
    container:
        "docker://python:3.11-slim"
    script:
        "plot.py"
```

That is a major advantage over trying to manage the whole workflow inside one
shared login environment.

{{< callout type="note" title="Global versus per-rule containers" >}}
Snakemake also allows a global container definition, but per-rule containers
are often clearer for teaching and for real workflows because they keep the
software requirements close to the rule that needs them.
{{< /callout >}}

## Practical Notes

The idea of containers is the same everywhere, but some execution details
depend on the system where you run the workflow.

- On a local machine, you need Apptainer installed on the system.
- On shared systems, Apptainer may already be available.
- External paths may need additional bind mounts.
- Exact execution details can depend on the local site configuration.

## A Small Reality Check

{{< challenge title="What if the container is wrong?" >}}

Change the container image for `make_plot` from
`docker://python:3.11-slim` to `docker://alpine:latest` and run the workflow
again.

What do you expect to happen, and why?

{{< solution >}}
The rule should fail, because `alpine:latest` does not provide the Python
environment needed to run `plot.py`.

That failure is actually useful: it shows that the job is really running inside
the declared container, not in your ordinary login environment.
{{< /solution >}}
{{< /challenge >}}

{{< instructor >}}
If learners ask about site-specific details such as extra bind mounts or why a
rule works on one system but not another, acknowledge that these are important
questions but keep the main focus on the general container idea first.
{{< /instructor >}}
