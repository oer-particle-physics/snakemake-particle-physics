+++
title = "Scaling with Wildcards and Scatter-Gather"
weight = 20
teaching = 15
exercises = 10
questions = [
  "How can one rule process many event files?",
  "What does a scatter-gather workflow look like in Snakemake?",
  "How do wildcards and `expand()` help a workflow scale?"
]
objectives = [
  "Use wildcards to run the same rule over many files.",
  "Use `expand()` to define a whole collection of expected outputs.",
  "Build a simple scatter-gather workflow.",
  "Use `script:` for a small gather step."
]
keypoints = [
  "**Wildcards** let one rule match many input and output files.",
  "**Scatter-gather** means running many independent jobs and then combining their outputs.",
  "**expand()** is a convenient way to define a collection of target files.",
  "**script:** is useful when a workflow step is more naturally written as a small Python script than as a shell one-liner."
]
+++

In particle physics, we rarely process just one file. A dataset is usually
split across many files, we run the same selection on each file, and then we
gather the results into a final summary or plot. This pattern is often called
**scatter-gather**.

In this episode, we use Snakemake wildcards to build that pattern in a simple
form. We will assume that we already know which datasets and file chunks exist.
In the next episode, we will look at what to do when that information is only
discovered at run time.

## Preparing the Input Files

Create a few toy inputs:

```bash
mkdir -p input/DYJets input/TTbar input/Data

for chunk in 0 1 2; do
    printf "Selected\nCutAway\nSelected\n" > "input/DYJets/DYJets.$chunk.txt"
    printf "Selected\nSelected\nCutAway\n" > "input/TTbar/TTbar.$chunk.txt"
    printf "CutAway\nSelected\nCutAway\n" > "input/Data/Data.$chunk.txt"
done
```

Each file stands in for one chunk of a larger dataset. Our workflow will:

1. run an event selection on every file
1. write one selected output per file
1. gather all selected files into one final summary

In real analyses, a dataset is often split across many files so that the work
can be processed in parallel. Here we use three small chunks per dataset, with
the identifiers `0`, `1`, and `2`. That is why the toy input files are named
like `DYJets.0.txt` and `TTbar.2.txt`.

## The Scatter Step

We begin by defining the datasets and file chunks we expect:

```python
DATASETS = ["DYJets", "TTbar", "Data"]
CHUNKS = ["0", "1", "2"]
```

We write the chunk identifiers as strings because Snakemake wildcards are
matched from file names. In a path such as `input/DYJets/DYJets.0.txt`, the
value of `{chunk}` is the text `"0"` taken from the file name.

Now we can write one generic rule for the selection step:

```python
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
```

{{< callout type="note" title="Why use `|| test $? -eq 1`?" >}}
The `grep` command returns exit code `0` when it finds at least one match, and
exit code `1` when it finds no matches. In this workflow, a file with no
selected events is not an error: it should simply produce an empty output file.

The extra `|| test $? -eq 1` tells the shell to treat that specific case as
successful. If `grep` fails for a real reason, such as a missing input file, it
will return a different exit code and the rule will still fail as it should.
{{< /callout >}}

This rule does not mention `DYJets`, `TTbar`, or `Data` explicitly. Instead, it
uses the wildcards `{dataset}` and `{chunk}`. Snakemake fills those values in by
matching the file names you ask it to create.

If Snakemake needs `selected/TTbar/TTbar.2.txt`, it infers:

- `dataset = TTbar`
- `chunk = 2`

and then runs the rule with the corresponding input file
`input/TTbar/TTbar.2.txt`.

## The Gather Step

After the selection, we want one final summary across all selected files. We
can define that with `rule all` and `expand()`:

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
    script:
        "plot.py"
```

The call to `expand()` generates the full list of selected files that we expect
to gather:

- `selected/DYJets/DYJets.0.txt`
- `selected/DYJets/DYJets.1.txt`
- `selected/DYJets/DYJets.2.txt`
- `selected/TTbar/TTbar.0.txt`
- and so on

This is the gather part of the workflow: one rule depends on many upstream
files and combines them into one output.

## Using `script:` for the Gather Logic

The final step is easier to read as a short Python script than as a long shell
command. Create a file called `plot.py`:

```python
from collections import OrderedDict
from pathlib import Path


counts = OrderedDict()

for input_file in map(Path, snakemake.input):
    dataset = input_file.parent.name
    counts.setdefault(dataset, 0)

    with open(input_file, "r", encoding="utf-8") as handle:
        counts[dataset] += sum(
            1 for line in handle if line.strip() == "Selected"
        )

output_path = Path(snakemake.output[0])
output_path.parent.mkdir(parents=True, exist_ok=True)

with open(output_path, "w", encoding="utf-8") as handle:
    for dataset, count in counts.items():
        handle.write(f"{dataset}\t{count}\n")
```

This script reads every selected file, counts the remaining events for each
dataset, and writes a small text summary.

{{< callout type="note" title="Why use script?" >}}
For short transformations, `shell:` is often enough. For slightly richer logic,
`script:` keeps the workflow readable and lets you write ordinary Python.
{{< /callout >}}

## Running the Workflow

Start with a dry-run:

```bash
pixi run snakemake -n -p
```

Then run the workflow with a few cores:

```bash
pixi run snakemake --cores 4
```

Snakemake can run the independent `select_events` jobs in parallel and then run
`make_plot` once all selected files are ready.

This is the key scaling idea:

- **scatter** across many independent files
- **gather** the results into a final output

## Adding Another Dataset

{{< challenge title="Add WJets" >}}

Add a new dataset called `WJets`.

1. Add `"WJets"` to the `DATASETS` list.
1. Create three input files:

   ```bash
   mkdir -p input/WJets

   for chunk in 0 1 2; do
       printf "Selected\nCutAway\nCutAway\n" > "input/WJets/WJets.$chunk.txt"
   done
   ```

1. Run a dry-run and then the workflow again.

Which jobs should Snakemake add to the workflow?

{{< solution >}}
Snakemake should add three new `select_events` jobs for the `WJets` files and
then rerun `make_plot`, because the final summary now depends on additional
inputs.

The existing `DYJets`, `TTbar`, and `Data` selection outputs do not need to be
rerun.
{{< /solution >}}
{{< /challenge >}}

## Looking Ahead

This episode assumed that we knew the datasets and chunk identifiers in
advance. That is often good enough, and it keeps the workflow simple.

Sometimes, however, the exact set of files is only known after an earlier step
has run. That is where Snakemake `checkpoint`s become useful, and that is the
topic of the next episode.
