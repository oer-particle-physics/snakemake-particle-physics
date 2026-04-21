+++
title = "Getting started"
weight = 10
teaching = 10
exercises = 10
questions = ["How do I install Snakemake?",
            "How do I run my first workflow?"]
objectives = ["Be able to install Snakemake and run a first workflow that does some useful work already."]
keypoints = ["Snakemake is hosted on `bioconda` and can be installed using `pixi`", ""]
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

<https://snakemake.readthedocs.io/en/stable/tutorial/basics.html>

{{< challenge title="Local vs shared" >}}
List two things that should stay in this lesson repository and two things that should stay in the shared module.

{{< hint >}}
Think about what is specific to your lesson topic versus what should stay reusable across many lessons.
{{< /hint >}}

{{< solution >}}
Lesson prose, schedule, glossary/profile content, and local branding should stay in this repository.
Shared layouts, pedagogy shortcodes, CSS/JS behavior, and validation tooling should stay upstream in `hugo-styles`.
{{< /solution >}}
{{< /challenge >}}

{{< instructor >}}
Point maintainers to the update guide early so they know updates should arrive through module version bumps:
[Updating Downstream Lessons](https://oer-particle-physics.github.io/hugo-styles/docs/updates/).
{{< /instructor >}}
