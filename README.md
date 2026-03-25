[English](README.md) | [繁體中文](README.zh.md)

# _ref-review-criteria

A reference checklist of code review criteria and quality standards for the Workshop codebase.

## Description

This reference skill provides a structured quality checklist used during code reviews in the Workshop project. It covers security, architecture, code quality, performance, and testing standards — injected as context by other skills and agents to ensure consistent review outcomes without duplicating criteria across multiple skill files.

## Features

- Security checklist: secrets, input validation, SQL injection, auth enforcement
- Architecture rules: module boundaries, EventBus-only cross-module writes, thin routes
- Code quality standards: function length limits, naming conventions, type hints
- Performance guidelines: N+1 prevention, pagination, Redis caching, embedding concurrency
- Testing requirements: hard-delete test data, idempotent event handlers

## Usage

This skill is a reference (`user-invocable: false`) and is not meant to be called directly. It is injected as context by reviewer agents and code-review skills.

## How It Works

The checklist is structured into five categories aligned with the Workshop architecture. When a reviewer agent or skill needs evaluation criteria, it reads this reference to apply consistent standards. Each category maps directly to a layer of the Workshop stack (security layer, module boundary rules, service layer patterns, data layer, test layer).

## Requirements

- Claude Code CLI
- Used as a reference by: `reviewer` agent, code-review skills

## License

MIT
