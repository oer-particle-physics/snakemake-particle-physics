+++
title = "Getting started"
weight = 10
teaching = 10
exercises = 10
questions = ["How do I start a lesson from this template?"]
objectives = ["Follow a reliable first-run order: setup, metadata update, then content replacement.", "Update the key `hugo.toml` fields so links, branding, and deployment targets are correct."]
keypoints = ["Start with `learners/setup`, then update `hugo.toml` before writing lesson content.", "Keep lesson prose local and framework behavior in the shared `hugo-styles` module.", "Use the shared `hugo-styles` docs after the local lesson skeleton is running."]
+++

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
