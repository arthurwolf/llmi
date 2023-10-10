# llmi

Large-Language-Model to Machine Interface Open-Source project.

## Goal

Large Language Models (LLMs) are very good at some things (writing, summarizing, answering factual questions), but also remarkably bad at others (real-time information, complex logic, information formatting, tool usage, verification, structured/long output).

Some of those things they are bad at, they could get better at if they were able to get help from tools/machines that currently 

This project aims to create:

* Specifications
* Training datasets
* Large language models
* And tools

in order to create LLMs with the ability to use computers to improve their abilities.

## Implementation

The plan here is, for each/all of the features, to:

1. Write down a specification of what the feature should be able to accomplish
2. How to accomplish/implement it.
3. Create a training dataset for this feature.
4. Once all datasets exist, train a LLM using these datasets.
5. Test and improve the LLM.
6. Write tools that are able to take advantage of the new features.

## Features

This is a presentation of each feature this project aims to implement.

PRs for extra features (and any contribution) are extremely welcome:

### Recursive redaction.
