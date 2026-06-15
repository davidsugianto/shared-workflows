# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repository is currently minimal: the only tracked project file is `README.md`, which describes the repo as a centralized place for reusable workflows.

There is no build configuration, package manifest, test runner, linter configuration, or application source tree checked in at the moment.

## Common commands

Because no language-specific tooling or project manifests are present yet, there are no repository-defined build, lint, or test commands to rely on.

Useful inspection commands while the repo is still being initialized:

- `git status`
- `git ls-tree -r --name-only HEAD`
- `find . -maxdepth 3 -type f | sort`

## Architecture

There is no implementation code in the repository yet.

At the moment, treat this repository as a placeholder for shared workflow definitions rather than an active application or library. Before making assumptions about runtime, tooling, or test strategy, inspect the current tree and any newly added manifests or CI configuration.

## Source of truth

- `README.md` is the only project documentation currently present.
- If workflow files, package manifests, or CI configuration are added later, update this file to document the actual development commands and repository structure.
