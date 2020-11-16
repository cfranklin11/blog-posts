---
title: How to Set up FaunaDB for local development
published: false
description:
tags: #python, #node, #serverless, #database
---

## What is FaunaDB, and why should I try it?

- Serverless DB
  - Compare to alternatives (DynamoDB, Aurora Serverless)
- FQL vs GQL API
- Link to quick start documentation

## Requirements

- Docker
- FaunaDB shell
- API testing tool (e.g. Postman, Insomnia) (optional, but easier than curl)
- Python (optional for integration tests)

## Setting up local FaunaDB

- Run in Docker container
- Create endpoint & database with FaunaDB shell
- Create API key with FaunaDB shell
- Import GQL schema via Postman
- Create record via Postman
- Query record via Postman

## Bonus: Connecting to FaunaDB for integration tests (Python & Node)

- Python
  - Use conftest to avoid using dev DB
  - Use pytest fixtures w/ client class to reset & use test DB
- Node
  - TBD
