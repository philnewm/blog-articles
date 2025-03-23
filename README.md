# Blog-Articles

## Description

Stores draft and published blog-articles together with their required resources.

## Markdown rules

- Articles title should be Header 1 (#)
- Place separators under chapter titles, Header 2 (##)
- Article should be valid markdown
- Specific terms should be formatted *italic*
- Use `sudo` for commands expected to be run as root

## Branch-States

| Article         | CI-State                                                                                                                                                                                                                                                 | GitHub-Pages                                                                                                                                                                                               | Dev.to                                                                                                                                                                                         |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| getting_started | [![getting-started-ci](https://github.com/philnewm/blog-articles/actions/workflows/ansible_molecule_getting_started_ci.yml/badge.svg?branch=draft)](https://github.com/philnewm/blog-articles/actions/workflows/ansible_molecule_getting_started_ci.yml) | [![Github Pages Pipeline](https://github.com/philnewm/blog-articles/actions/workflows/gh_pages_pipeline.yml/badge.svg)](https://github.com/philnewm/blog-articles/actions/workflows/gh_pages_pipeline.yml) | [![Dev.to Pipeline](https://github.com/philnewm/blog-articles/actions/workflows/devto_pipeline.yml/badge.svg)](https://github.com/philnewm/blog-articles/actions/workflows/devto_pipeline.yml) |
