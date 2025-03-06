# Plan for Merging Docker Images

This document outlines the plan for merging the `nlp/Dockerfile` and `minimal/Dockerfile` Docker images.

## Goal

Create a single Docker image suitable for testing NLP methods using Ruby in a Jupyter Notebook.

## Base Image

`quay.io/jupyter/minimal-notebook`

## Steps

1.  **Base Image:** Start with the `quay.io/jupyter/minimal-notebook` base image.
2.  **Gems:** Use the gems from `nlp/Gemfile`.
3.  **Python Packages:** Install the Python packages listed in the `nlp/Dockerfile` using `mamba` and `pip`.
4.  **Patch:** Copy and apply the `respond_to_missing.patch` from the `nlp/Dockerfile`.
5.  **Environment Variables:** Set the environment variables from both Dockerfiles, resolving any conflicts.
6.  **User and Permissions:** Ensure the user and permissions are correctly set up for the Jupyter Notebook environment.
7.  **iruby Kernel:** Register the iruby kernel.

## Dockerfile Structure

```mermaid
graph LR
    A[FROM quay.io/jupyter/minimal-notebook] --> B(Install system dependencies from minimal/Dockerfile);
    B --> C(Set environment variables from minimal/Dockerfile);
    C --> D(Create user and set permissions from minimal/Dockerfile);
    D --> E(Copy Ruby from builder stage of minimal/Dockerfile);
    E --> F(Copy installed gems from builder stage of minimal/Dockerfile);
    F --> G(Install Python packages from nlp/Dockerfile using mamba and pip);
    G --> H(Copy nlp/Gemfile and run bundle install);
    H --> I(Copy and apply respond_to_missing.patch from nlp/Dockerfile);
    I --> J(Set environment variables from nlp/Dockerfile);
    J --> K(Register iruby kernel);