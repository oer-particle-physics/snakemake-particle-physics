+++
title = "Getting started"
weight = 10
teaching = 10
exercises = 10
questions = [
  "How do I run my first workflow?",
  "How do I define a single processing step?",
  "How do I execute a rule?"
]
objectives = [
  "Be able to create a `Snakefile`.",
  "Understand the contents of a basic Snakemake `rule` with `input`, `output`, and `shell`.",
  "Be able to execute a specific workflow file."
]
keypoints = [
  "A `Snakefile` defines the workflow.",
  "A `rule` contains `input`, `output`, and a `shell` command",
  "You execute the workflow by asking for the **output file**, not the rule name."
]
+++

From the [Snakemake homepage](https://snakemake.readthedocs.io/en/stable/index.html):

> The Snakemake workflow management system is a tool to create
> reproducible and scalable data analyses.
> Workflows are described via a human readable, Python based language.
> They can be seamlessly scaled to server, cluster, grid and cloud environments,
> without the need to modify the workflow definition.
> Snakemake workflows can entail a description of required software,
> which will be automatically deployed to any execution environment.
> Finally, workflow runs can be automatically turned into interactive
> portable browser based reports,
> which can be shared with collaborators via email or the cloud
> and combine results with all used parameters, code, and software.

## The Essentials

{{< callout type="note" title="Acknowledgements" >}}
The following material leans heavily on a
[tutorial by Alejandro Gomez](https://alefisico.github.io/snakemake-cms-tutorial/01-intro.html).
{{< /callout >}}

To run a workflow, you need two things:

1. **The Snakefile**: A file named `Snakefile` (capital S, no extension) where you define your rules.
  It uses a Python-based syntax, so you can write standard Python code inside it alongside your rules.
2. **The Execution Command**: You run the workflow from the terminal by invoking `snakemake`.

The basic command structure is:

```bash
snakemake --cores <N> <target_file>
```

- `--cores <N>`: Tells Snakemake how many CPU cores to use.
  (e.g. `--cores 1` for sequential execution, `--cores 4` for parallel).
- `<target_file>`: The file you want to generate. Snakemake will figure out the steps to get there.
- If you dont specify a `<target_file>`,
  Snakemake will look for a file named `Snakefile` in the current directory.
  You can specify a different file with `-s <filename>`.

---

## The Anatomy of a Rule

The building block of Snakemake is the `rule`.
Think of it as a recipe.
To make a dish (output), you need ingredients (inputs),
a kitchen (the environment),
and a set of instructions (the shell command).

```python
rule skim_data:
    input:
        "raw_data.txt"
    output:
        "skimmed_data.txt"
    shell:
        "grep 'Signal' {input} > {output}"
```

Important Properties:

- **Rule Name**: Must be unique (here: `skim_data`).
- **Input/Output**: These are strings (or lists of strings).
Snakemake uses these to "connect the dots" between rules.
- **Shell**: The bash command to execute.
  Note the `{input}` and `{output}` placeholders—Snakemake automatically fills these in with the paths you defined above.

### Understanding the Syntax

A `Snakefile` is essentially a Python script with some extra keywords added.
This is powerful because it means you can use Python variables, functions,
and libraries directly in your workflow definition.

- **Indentation matters**: Just like in Python, Snakemake uses indentation (usually 4 spaces) to group code blocks. The `input`, `output`, and `shell` directives must be indented relative to the `rule`.
- **Strings**: File paths and commands are strings, so they must be enclosed in quotes (`"..."` or `'...'`).
- **Comments**: Use `#` for comments, just like in Python/Bash.
- **Lists**: If a rule has multiple inputs or outputs, you can list them with commas:

```python
    input:
        "file1.txt",
        "file2.txt"
```

## Running the Rule

1. Create dummy data:

  ```bash
  echo -e "Background\nSignal\nBackground\nSignal" > raw_data.txt
  ```
1. Create a `Snakefile` with the content shown above.

1. Run Snakemake. We must tell it **what file we want to generate**:

```bash
pixi run snakemake --cores 1 skimmed_data.txt
# if you run snakemake from a conda environment or pip, use:
# snakemake --cores 1 skimmed_data.txt
```

If successful, you will see `Finished jobid: 0`.

{{< solution title="Command output" >}}

```text
Assuming unrestricted shared filesystem usage.
host: gluon
Building DAG of jobs...
Using shell: /usr/bin/bash
Provided cores: 1 (use --cores to define parallelism)
Rules claiming more threads will be scaled down.
Job stats:
job          count
---------  -------
skim_data        1
total            1

Select jobs to execute...
Execute 1 jobs...

[Sun Feb  8 12:05:08 2026]
localrule skim_data:
    input: raw_data.txt
    output: skimmed_data.txt
    jobid: 0
    reason: Missing output files: skimmed_data.txt
    resources: tmpdir=/tmp
[Sun Feb  8 12:05:08 2026]
Finished jobid: 0 (Rule: skim_data)
1 of 1 steps (100%) done
```

{{</ solution >}}

{{< instructor >}}
If `snakemake` was not installed using pixi but with a conda environment or pip, you should remove the `pixi run` part and just run:

```bash
snakemake --cores 1 counts.txt
```

This will be the case for most users following this tutorial outside of the pixi environment.
{{</ instructor >}}

{{< challenge title="Syntax Error Hunt" >}}

Intentionally break your indentation (remove a space before `input:`).
Run the command again.

What error does Snakemake give you?

{{< solution >}}
An `IndentationError` is the most common error you will encounter.
{{< /solution >}}
{{< /challenge >}}
