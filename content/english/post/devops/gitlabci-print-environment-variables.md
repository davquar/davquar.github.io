---
author: Davide Quaranta
title: Can you print the configured environment variables in GitLab CI/CD pipelines?
date: 2023-05-22T23:00:00+02:00
categories: [DevOps]
tags: [gilabci, cicd]
description: "We had the need to print the environment variables used as parameters of GitLab CI/CD pipelines. Right now there is no official straightforward way to do it, so I resorted to a tiny custom way to do it."
---

For both debugging and "explanability" reasons it is very handy to know which parameters were used to start GitLab CI/CD pipelines.

Jenkins, for examples, does exactly this out of the box. GitLab CI/CD is super nice but doesn't have this feature at the moment.

So I eventually crafted a solution for that.

## The situation

First of all, the anatomy of our `.gitlab-ci.yml` is the following:

```yaml
todo: todo
```

Hence the following rules:

- Each pipeline has a configuration of environment variables, defined in the `variables` section.
- At the moment we aren't interested in other non-defined variables.

The problem is that a simple job that print the `env` also reveals other environment variables that are not interesting. So we need something more complex.

## Some ideas

I explored these ideas:

### Idea 1: the "diff"

Exploiting the [`inherit:variables`](https://docs.gitlab.com/ee/ci/yaml/#inheritvariables) option it is possible to get the difference between all the environment variables and only the ones that are injected by GitLab.

In other (totally not needed) words, let's define:

- $A$: the set of all environment variables of the job.
- $D$: the set of the default injected variables.

Then, $X = A - D$ is the unknown set of the defined variables (what we want).

This idea looks perfect on paper, but doesn't work as expected, because you actually need two jobs to achieve it:

- Job 1: calculate $A$ with `env`.
- Job 2: calculate $D$ with `env` and `inherit:variables` at `false`.

And in each job you get different values for some default variables that GitLab injects, so a simple `diff` wouldn't work.

(yes, you could work more on that path, for example by having a predefined list of unwanted default variables to filter out, but I didn't want to hardcode anything)

### Idea 2: YAML tricks

The idea is to fully exploit the characteristics of the YAML language to print the very `variables` dictionary.

One way is to define an anchor to `variables` and somehow `echo` it, but I didn't find a way to do it.

### Idea 3: little custom solution

The images that we use to run the jobs contain some utilities, including [`yq`](https://github.com/mikefarah/yq), which allows to parse and transform YAML.

So the idea is:

1. Parse the very `.gitlab-ci.yml` file to get a list of all the defined variable names.
2. Flatten the result into a string representing a series of `OR` that `grep` can evaluate.
3. Use that expression to `grep` the result of `env`.

You can do this with a one-liner in a single job.

```yaml
todo: todo
```

