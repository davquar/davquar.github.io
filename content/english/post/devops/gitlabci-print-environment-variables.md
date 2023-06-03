---
author: Davide Quaranta
title: Printing environment variables in GitLab CI/CD pipelines
date: 2023-06-03T00:12:00+02:00
categories: [DevOps]
tags: [gilabci, cicd]
description: "We had the need to print the environment variables used as parameters of GitLab CI/CD pipelines. Right now there is no official straightforward way to do it, so I resorted to a tiny custom way to do it."
---

For both debugging and "explanability" reasons it is very handy to know which parameters were used to start GitLab CI/CD pipelines.

Jenkins, for examples, does exactly this out of the box. GitLab CI/CD is super nice but doesn't have this feature at the moment.

So I eventually crafted a solution for that.

## The situation

First of all, the head of our `.gitlab-ci.yml` is the following:

```yaml
variables:
    VARIABLE_1:
        value: "default value"
        description: "This variable is used to do x"

    VARIABLE_2:
        value: ""
        description: "This variable is used to do y"

    ...
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

Then, $X = A - D$ is the set of the defined variables (what we want).

This idea looks perfect on paper, but doesn't work as expected, because you actually need two jobs to achieve it:

- Job 1: calculate $A$ with `env`.
- Job 2: calculate $D$ with `env` and `inherit:variables` to `false`.

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
print_variables:
    stage: .pre
    script:
        - env | egrep $(yq '.variables | keys | ... comments = "" | map("^" + . + "=") | @csv' .gitlab-ci.yml | tr "," "|")
```

Let's unpack it and explain it:

1. First get the `env`.
2. Pipe it to the result of a `grep`:
   1. Parse the `variables` section of the YAML file.
   2. Just keep the keys.
   3. Remove comments.
   4. Map every `VAR` to `^VAR=`, to create a regex to match any string that starts with `VAR` and ends with `=`.
   5. Transform all to a CSV string (`VAR1,VAR2,...`).
   6. Replace commas with pipes, to complete the regex (`VAR1|VAR2|...`).

The result is just the portion of `env` which keys are defined in the `variables` section.

## Conclusion

Do I like this solution? Yes and no. Is it perfect? Nope.
I would have preferred something given by GitLab.
Maybe there is a more elegant solution but I don't know it.
In any case, this solution works good, is fast, compact and yes, also elegant.