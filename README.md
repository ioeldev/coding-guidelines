# Coding Guidelines

Personal TypeScript and React coding conventions, with Bun as the default runtime and package manager for new projects. Existing project tooling is respected.

## Install

```sh
bunx skills add ioeldev/coding-guidelines
```

Or with npm:

```sh
npx skills add ioeldev/coding-guidelines
```

Choose your coding agents when prompted. Installation is project-local by default; append `-g` to make the skill available across projects.

To install globally for Codex:

```sh
bunx skills add ioeldev/coding-guidelines --skill coding-guidelines -a codex -g
```

## Guidelines

The skill covers descriptive naming, flat control flow, logical spacing, focused functions, domain-based architecture, strict TypeScript, React patterns, promises and errors, and Bun tooling.

Read [SKILL.md](skills/coding-guidelines/SKILL.md) for the full instructions.

## Usage

Ask your agent to use the coding-guidelines skill when writing, reviewing, or refactoring code. Automatic selection depends on the agent and the task; a skill does not guarantee the always-loaded behavior of AGENTS.md. In Codex, invoke it explicitly with `$coding-guidelines`.

For a project-level reminder, add this short instruction to AGENTS.md:

> Use the coding-guidelines skill when writing, reviewing, or refactoring code.

## Updates

```sh
bunx skills update
```

No npm package or custom installer is required. Installation is handled by the [skills CLI](https://github.com/vercel-labs/skills).
