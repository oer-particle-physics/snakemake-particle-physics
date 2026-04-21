+++
title = "Getting Started: Rules, Targets, and the DAG"
weight = 10
teaching = 15
exercises = 10
questions = [
  "How do I run my first Snakemake workflow?",
  "How does Snakemake connect rules together?",
  "How does Snakemake decide what needs to run again?"
]
objectives = [
  "Create a minimal `Snakefile` with `input`, `output`, and `shell`.",
  "Run Snakemake by asking for an output file.",
  "Use `rule all` to define the default workflow target.",
  "Understand dry-runs and lazy re-execution."
]
keypoints = [
  "A workflow is defined in a `Snakefile`.",
  "Snakemake works backwards from the output you request.",
  "Rules are connected by matching outputs to inputs.",
  "`rule all` defines the default target for the workflow.",
  "Snakemake only reruns outputs that are missing or stale."
]
+++

In this episode, we build a small event-selection workflow and use it to
introduce the main Snakemake idea: you ask for the output you want, and
Snakemake works out the steps needed to create it.

{{< callout type="note" title="Acknowledgements" >}}
This lesson started from a tutorial by
[Alejandro Gomez](https://alefisico.github.io/snakemake-cms-tutorial/01-intro.html)
and has since been adapted for this course.
{{< /callout >}}

## A First Rule

Create a small input file:

```bash
cat <<'EOF' > events.txt
Background
Signal
Background
Signal
EOF
```

Now create a `Snakefile` with one rule:

```python
rule select_events:
    input:
        "events.txt"
    output:
        "selected_events.txt"
    shell:
        "grep 'Signal' {input} > {output}"
```

This rule says:

- the input is `events.txt`
- the output is `selected_events.txt`
- the command that transforms one into the other is a shell command

## Running by Target

Run Snakemake by asking for the output file you want:

```bash
pixi run snakemake --cores 1 selected_events.txt
```

Snakemake looks at the requested target, finds a rule that can create it, and
then runs that rule because `selected_events.txt` does not exist yet.

{{< callout type="note" title="Rule Names and Targets" >}}
The rule is called `select_events`, but we do not run it by name. We ask for
the file we want, here `selected_events.txt`.
{{< /callout >}}

## Thinking Backwards

Now extend the workflow with a second rule:

```python
rule select_events:
    input:
        "events.txt"
    output:
        "selected_events.txt"
    shell:
        "grep 'Signal' {input} > {output}"


rule count_events:
    input:
        "selected_events.txt"
    output:
        "event_counts.txt"
    shell:
        "wc -l {input} > {output}"
```

If you now run

```bash
pixi run snakemake --cores 1 event_counts.txt
```

Snakemake reasons backwards:

1. You want `event_counts.txt`.
1. `count_events` can create it, but it needs `selected_events.txt`.
1. `select_events` can create `selected_events.txt` from `events.txt`.

This chain of dependencies is the **Directed Acyclic Graph**, or **DAG**. In a
larger analysis, the same idea scales from two short rules to hundreds of jobs.

## Defining a Default Target

If you do not specify a target on the command line, Snakemake uses the first
rule in the `Snakefile`. By convention, we make that first rule a rule called
`all`, which collects the final outputs we care about.

Update your `Snakefile` to:

```python
rule all:
    input:
        "event_counts.txt"


rule select_events:
    input:
        "events.txt"
    output:
        "selected_events.txt"
    shell:
        "grep 'Signal' {input} > {output}"


rule count_events:
    input:
        "selected_events.txt"
    output:
        "event_counts.txt"
    shell:
        "wc -l {input} > {output}"
```

Now you can simply run:

```bash
pixi run snakemake --cores 1
```

## Dry-Runs and Lazy Re-Execution

Before running a workflow, it is often useful to ask Snakemake what it would
do without actually executing anything:

```bash
pixi run snakemake -n -p
```

The `-n` flag performs a dry-run, and `-p` prints the shell commands.

If you run the workflow again immediately afterwards, Snakemake should report
that nothing needs to be done, because all requested outputs are present and up
to date.

This is one of the most useful features of Snakemake: it does not rerun
everything blindly. It only reruns work when an output is missing or when one
of its inputs has changed.

{{< challenge title="What Will Re-Run?" >}}

1. Update the timestamp of the original input:

   ```bash
   touch events.txt
   ```

2. Run a dry-run:

   ```bash
   pixi run snakemake -n -p
   ```

Which rules does Snakemake want to rerun, and why?

{{< solution >}}
Snakemake should want to rerun both `select_events` and `count_events`.

Once `events.txt` becomes newer than `selected_events.txt`, the selected file is
considered stale. Since `event_counts.txt` depends on `selected_events.txt`, it
also becomes stale and must be recreated.
{{< /solution >}}
{{< /challenge >}}

{{< instructor >}}
If learners are not using the Pixi environment, they can remove `pixi run` and
run `snakemake` directly. To keep the lesson readable, the main text uses the
Pixi-based commands only.
{{< /instructor >}}
