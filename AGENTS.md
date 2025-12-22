# AGENTS.md

## Build/Lint/Test Commands

This is a minimal LMS plugin with no automated tests. No build/lint/test commands exist.

## Code Style Guidelines

### Perl (Plugin.pm)
- Keep minimal - only required `initPlugin {}` function
- Use standard Perl package declaration: `package Plugins::MobileTranscode::Plugin;`
- End file with `1;` as required by Perl modules

### XML Files (install.xml, repo.xml)
- Follow existing indentation (2 spaces)
- Maintain exact structure for LMS compatibility
- Version format: X.Y.Z (semantic versioning)

### custom-convert.conf
- Use tabs for indentation (LMS requirement)
- Format: `[source] [dest] [model] [player]`
- Comments start with `#`
- Capability flags in comments: I/F/T/U/D
- Pipeline commands use `[tool]` syntax with LMS variables

### Naming Conventions
- Model names: PascalCase (e.g., LyrPlay, YourApp)
- Files: kebab-case for README, lower-case for config
- Plugin namespace: Plugins::MobileTranscode::Plugin

### Critical Rules
- SHA1 in repo.xml must exactly match release zip
- Always update install.xml version first before release
- Test with actual mobile clients - no automated validation available