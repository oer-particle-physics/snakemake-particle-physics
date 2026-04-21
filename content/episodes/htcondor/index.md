+++
title = "Running on HTCondor"
weight = 40
teaching = 10
exercises = 0
questions = [
  "How do I run the same Snakemake workflow on HTCondor?",
  "What should go into a workflow profile?",
  "Why do resources and batching matter on HTCondor?"
]
objectives = [
  "Understand how local and batch execution can use the same `Snakefile`.",
  "Use a workflow profile to store HTCondor-specific execution settings.",
  "Recognise which resource settings matter for HTCondor jobs.",
  "Know where to find a concrete HTCondor example for further study."
]
keypoints = [
  "The `Snakefile` should usually stay the same across local and batch execution.",
  "A workflow profile is the clean place to store HTCondor-specific executor settings.",
  "Resource requests and batching strategy strongly affect queue efficiency.",
  "The native HTCondor executor plugin avoids older submission wrappers."
]
+++

After a workflow runs locally, the next step is often to submit it to an
HTCondor cluster. The main idea is simple: the workflow logic should stay the
same. In most cases, you should not rewrite rules for HTCondor. Instead, you
change how Snakemake is executed.

This is a short reference episode rather than a full live exercise. The
concrete example used here comes from CERN, and lives in the
[`snakemake-lxplus-example` repository](https://github.com/clelange/snakemake-lxplus-example/tree/htcondor_plugin)
(and will probably soon be moved under the
[`hep-workflows` GitHub organization](https://github.com/hep-workflows)).

## Same Workflow, Different Execution

The same rule can often run:

- locally on your machine
- on a login node
- on an HTCondor cluster such as LXBATCH

That is one of the strengths of Snakemake. The rule still declares its
`input`, `output`, resources, and software environment. What changes is the
executor and the profile that Snakemake uses at run time.

{{< callout type="note" title="CERN-specific note" >}}
In the CERN examples, it is usually better to run workflows from your **AFS** work area
rather than your home directory.
{{< /callout >}}

## Install the Executor Plugin

In addition to `snakemake` itself, HTCondor execution needs the corresponding
executor plugin and the HTCondor Python bindings. If you are working in the
same Pixi environment that you created during setup, add them with:

```bash
pixi add python-htcondor snakemake-executor-plugin-htcondor
```

After that, `pixi run snakemake` can use the HTCondor executor from the same
environment.

## Use a Workflow Profile

For HTCondor execution, the cleanest approach is to keep cluster-specific
settings in a workflow profile and then run Snakemake with:

```bash
pixi run snakemake --workflow-profile lxbatch
```

For Snakemake 9 and later, a minimal profile can be saved as
`workflow/profiles/lxbatch/profile.v9+.yaml`:

{{< details title="Minimal HTCondor profile" >}}
```yaml
executor: htcondor
jobs: 5000
local-cores: 10
htcondor-jobdir: .condor_jobs
default-resources:
  - getenv=True
  - htcondor_request_mem_mb=1024
  - htcondor_request_disk_mb=1024
  - classad_JobFlavour=espresso
```
{{</ details >}}

This keeps the workflow itself readable. The `Snakefile` describes the analysis,
while the profile describes how jobs should be submitted on the cluster.

## A Minimal HTCondor Example

With that profile in place, a minimal `Snakefile` could look like:

{{< details title="Minimal HTCondor Snakefile" >}}
```python
rule all:
    input:
        "local_hello.txt"


rule hello_htcondor:
    output:
        "local_hello.txt"
    shell:
        "echo Hello from $(hostname) > {output}"
```
{{</ details >}}

You can then run:

```bash
pixi run snakemake --workflow-profile lxbatch --cores 1
```

This uses the native `snakemake-executor-plugin-htcondor` and submits a simple
rule through HTCondor.

The important point is not the particular example rule. The important point is
that it is still an ordinary Snakemake rule. Batch execution is selected at run
time by the workflow profile.

{{< callout type="note" title="Optional: a Pixi task" >}}
If you want, you can add the following snippet to the generated `pixi.toml`:

{{< details title="Optional Pixi task snippet" >}}
```toml
[tasks]
snakemake-htcondor = "snakemake --workflow-profile lxbatch --cores 1"
```
{{</ details >}}

You can then run:

```bash
pixi run snakemake-htcondor
```
{{< /callout >}}

## Resources

When jobs run on LXBATCH, resource requests become much more important than they
are in a small local example. At minimum, think about:

- memory
- disk
- runtime

It is often sensible to set conservative defaults in the profile and then
override them for unusually heavy rules in the workflow itself.

{{< details title="CERN-specific storage examples" >}}
The CERN example repository also includes EOS examples:

- writing directly to EOS from a batch job
- reading a file from EOS and copying the result back into the workflow directory

These examples are useful because they show that storage handling can still be
expressed as ordinary workflow inputs and outputs, rather than as ad hoc manual
steps.
{{</ details >}}

## Why Batching Matters

On batch systems, the right unit of work is not always “one input file per job”.
If the per-file tasks are very small, submitting one HTCondor job for each file
can create unnecessary overhead.

The example repository therefore includes two batching patterns:

- fixed-size batching
- cost-aware batching

Fixed-size batching is simpler, but cost-aware batching is often better when
some samples are known to run much longer than others.

{{< instructor >}}
Keep this episode short in a live workshop. The main goal is to show that
HTCondor execution changes the run configuration more than the workflow logic.
Leave the full batching and site-specific details for offline study in the
example repository linked at the top of the page.
{{< /instructor >}}
