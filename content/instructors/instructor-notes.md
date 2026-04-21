+++
title = "Instructor Notes"
weight = 10
audience = "instructor"
+++

## Lesson Structure

The lesson has five main episodes and one bonus episode:

1. `Getting Started: Rules, Targets, and the DAG`
1. `Scaling with Wildcards and Scatter-Gather`
1. `Dynamic Scatter-Gather with Checkpoints`
1. `Containers`
1. `Running on HTCondor`
1. `Bonus: Visualising the Workflow`

The main story stays close to a realistic analysis pattern:

- start from a simple workflow
- scale to many files with wildcards
- introduce scatter-gather
- show when dynamic discovery needs `checkpoint`
- show how software environments and batch execution fit around the same
  workflow logic

## Recommended 90-Minute Live Path

For a 1.5 hour session, do not try to teach every page in full detail. A good
live path is:

1. `Getting Started: Rules, Targets, and the DAG`
1. `Scaling with Wildcards and Scatter-Gather`
1. `Dynamic Scatter-Gather with Checkpoints`
1. a shorter pass through `Containers`
1. a short wrap-up with `Running on HTCondor`

Treat `Bonus: Visualising the Workflow` as optional or offline material.

## Suggested Timing

- `0-10 min`: motivation, setup check, and the “ask for outputs, not rule names”
  mindset
- `10-25 min`: first workflow, `rule all`, dry-runs, and reruns
- `25-45 min`: wildcards, `expand()`, and the main scatter-gather example
- `45-60 min`: checkpoints as a dynamic extension of the same example family
- `60-75 min`: containers, `container:`, and `--sdm apptainer`
- `75-85 min`: HTCondor profiles, executor plugin, and resource requests
- `85-90 min`: recap, questions, and pointers to bonus/offline material

If setup has to happen during the session, shorten the container and HTCondor
parts and treat them mainly as orientation.

## How to Use Each Episode

### Getting Started

Use this as the first real hands-on episode.

Main ideas:

- `Snakefile`
- `rule`, `input`, `output`, `shell`
- asking for an output file
- `rule all`
- dry-runs and lazy re-execution

Teaching note:

- learners often try to run a rule name rather than request an output file
- the “works backwards from the target” idea is the key conceptual shift

### Scaling with Wildcards and Scatter-Gather

This is the main practical episode.

Main ideas:

- wildcards
- `expand()`
- scatter-gather
- `--cores`
- `script:`

Teaching note:

- this is where the particle-physics framing really helps
- the example is intentionally simple, but it looks like a realistic
  file-by-file selection workflow
- the `CHUNKS = ["0", "1", "2"]` list works best if you explicitly remind
  learners that these are file identifiers coming from filenames

### Dynamic Scatter-Gather with Checkpoints

Teach this as a short guided demo rather than a long coding exercise.

Main ideas:

- when ordinary wildcards are enough
- when a file list is only known at run time
- `checkpoint`
- `checkpoints.<name>.get()`
- dynamic DAG expansion

Teaching note:

- the episode has a useful intermediate milestone: first produce all manifest
  files, then inspect one of them, and only afterwards show the full dynamic
  workflow
- prefer the explicit manifest-reading pattern in class
- keep `glob_wildcards()` as an instructor-side aside, not as the default
  learner-facing pattern

### Containers

This should stay in the main lesson, but it does not need to become a long
infrastructure detour.

Main ideas:

- `container:`
- `--software-deployment-method apptainer` / `--sdm apptainer`
- `APPTAINER_CACHEDIR`
- per-rule software environments

Teaching note:

- explicitly point out that `container:` does not activate itself; learners
  need to run Snakemake with `--sdm apptainer`
- the episode explains what happens if that option is omitted
- the cache-directory advice matters in practice, especially on shared systems
- the optional Pixi-task note is useful, but should stay a sidenote

### Running on HTCondor

Treat this as a short reference-style wrap-up, not as a full live exercise.

Main ideas:

- install the HTCondor executor plugin in the Pixi environment
- use a workflow profile
- keep the `Snakefile` largely unchanged
- request sensible resources
- think about batching

Teaching note:

- the page is deliberately self-contained, with copy-paste snippets for the
  profile, a minimal `Snakefile`, and an optional Pixi task
- the CERN example repository is linked at the top, but the main page does not
  depend on learners chasing links
- this page should show where the workflow goes next, not turn into a long
  HTCondor tutorial

### Bonus: Visualising the Workflow

Keep this as bonus or offline material.

Main ideas:

- `--dag`
- `--rulegraph`
- dry-runs as the safest first preview
- Graphviz output

Teaching note:

- useful for learners who want a workflow picture
- not essential to the main learning arc
- if time is short, this is the first episode to skip

## Offline or Optional Material

The following topics are better suited to self-study or follow-up discussion:

- Graphviz-based workflow visualisation
- more detailed HTCondor and EOS examples
- site-specific container and bind-mount details
- real-analysis integration and patching of external repositories

## Features Covered in the Main Path

The lesson covers several important Snakemake features in the main path:

- `rule all`
- dry-runs
- wildcards
- `expand()`
- `script:`
- `checkpoint`
- `container:`
- workflow profiles
- HTCondor execution with the native plugin

## Topics Worth Mentioning Briefly

Two features are only lightly covered and are worth mentioning in class, even
if they do not need their own full episode:

- `threads:`
- per-rule `resources:`

These fit naturally into the container or HTCondor discussion, because that is
where learners start asking about CPU, memory, disk, and runtime.

## Learner Trouble Spots to Watch

- Students often ask for a rule name instead of an output file.
- Students often place wildcards in `input` that are not determined by
  `output`.
- Students only really appreciate `checkpoint` after first seeing a case where
  ordinary wildcards and `expand()` are enough.
- Learners may think `container:` automatically enables Apptainer execution.
- Too much infrastructure detail too early makes Snakemake feel harder than it
  is.

## Teaching Note: Checkpoints and `glob_wildcards()`

In the checkpoint episode, prefer the explicit manifest-reading pattern as the
main learner-facing example.

Why this works better for teaching:

- it uses the actual output of the checkpoint
- it makes the data flow more explicit
- it preserves any filtering, ordering, or bookkeeping done by the checkpoint
- it avoids making the checkpoint feel like “magic filesystem rescanning”

`glob_wildcards()` is useful in real workflows, especially when the
checkpoint materialises a directory whose contents should themselves be treated
as truth. However, it is less explicit because it rescans the filesystem for
matching paths rather than reading the checkpoint output directly.

If you mention `glob_wildcards()` in class, frame it as a convenience pattern,
not as the default or preferred pattern for this lesson.
