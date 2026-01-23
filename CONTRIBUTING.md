# Contributing to OSMF

Thank you for your interest in contributing to the Open Study Model Framework. This document outlines guidelines for contributing to the project.

## Ways to Contribute

### 1. Report Issues

- **Spec ambiguities** — If something in the specification is unclear, open an issue
- **Schema bugs** — Report validation issues or schema errors
- **Suggestions** — Propose improvements or new features

### 2. Improve Documentation

- Fix typos or clarify wording in `OSMF-def.md`
- Add examples to help implementers understand concepts
- Improve schema descriptions

### 3. Propose Spec Changes

For changes to the core specification:

1. Open an issue describing the proposed change
2. Include rationale and use cases
3. Wait for discussion before submitting a PR

## Style Guidelines

### File Naming

- Use **kebab-case** for all files: `my-example.json`
- Schema files use `.schema.json` extension
- Example files use `.example.json` extension

### JSON Style

- 2-space indentation
- Use `camelCase` for property names within schemas
- Include `description` fields for all properties

### Documentation

- Use clear, concise language
- Prefer active voice
- Include examples where helpful

## Submitting Changes

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-change`
3. Make your changes
4. Ensure schemas are valid JSON
5. Use clear, descriptive commit messages
6. Submit a pull request with a clear description

## Review Process

1. All changes require review before merging
2. Spec changes require broad consensus
3. Schema changes must maintain backward compatibility where possible

## Versioning

OSMF follows semantic versioning:

- **MAJOR** — Breaking changes to the spec or schemas
- **MINOR** — New features, backward-compatible additions
- **PATCH** — Bug fixes, documentation improvements

## Code of Conduct

- Be respectful and constructive
- Focus on the technical merits of proposals
- Welcome newcomers and help them contribute

## Questions?

Open an issue :D

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (Apache 2.0).
