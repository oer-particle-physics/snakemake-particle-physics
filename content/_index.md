+++
title = "Snakemake for Particle Physics"
layout = "hextra-home"
+++

<div class="hx:mt-6 hx:mb-6">
{{< hextra/hero-headline >}}
{{< lesson/meta "title" >}}
{{< /hextra/hero-headline >}}
</div>

<div class="hx:mb-12">
{{< hextra/hero-subtitle >}}
{{< lesson/meta "tagline" >}}
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx:mb-6">
{{< hextra/hero-button text="Setup" link="learners/setup/" >}}
{{< hextra/hero-button text="Get Started" link="episodes/getting-started/" >}}
</div>


<div class="hx:mt-6"></div>
{{< hextra/feature-grid cols="3" >}}
{{< hextra/feature-card
  title="Scatter-Gather Workflows"
  subtitle="Use wildcards and `expand()` to run the same selection across many event files and then gather the results."
  icon="book-open"
  link="episodes/scaling/"
>}}
{{< hextra/feature-card
  title="Dynamic File Discovery"
  subtitle="Learn when ordinary wildcards are not enough and how `checkpoint` lets Snakemake expand the workflow at run time."
  icon="sparkles"
  link="episodes/checkpoints/"
>}}
{{< hextra/feature-card
  title="Containers and Environments"
  subtitle="Use per-rule containers to keep software requirements explicit and reproducible."
  icon="academic-cap"
  link="episodes/containers/"
>}}
{{< /hextra/feature-grid >}}

<div class="hx:mt-6"></div>
{{< lesson/overview >}}
<div class="hx:mt-6"></div>
{{< lesson/schedule title="Schedule" >}}
<div class="hx:mt-6"></div>
{{< lesson/authors title="Authors and Contributors" >}}
