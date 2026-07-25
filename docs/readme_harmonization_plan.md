# README harmonization plan

This backlog defines how to make the repository's README files consistent, useful, and
maintainable. It covers active entry points first, then specialized and historical areas.

## Naming rule

- The canonical simulator and product name is **LoRaFlexSim**.
- Never derive the product name from a checkout folder, archive suffix, release number, or
  local path.
- Keep the Python package and command names lowercase and unchanged: `loraflexsim`.
- Use repository-relative paths in documentation and committed examples so that instructions
  work from a Windows 11, Linux, or macOS checkout.

## Priority 0 — canonical public surface

- [x] Replace the obsolete suffixed product name in the tracked documentation.
- [x] Add an automated documentation check that rejects the obsolete product name.
- [ ] Rewrite the root overview around three user goals: discover, run, and reproduce.
- [ ] Remove duplicated quick-start sections and retain one Windows 11-first installation path.
- [ ] Verify every root command against the installed CLI and dashboard entry points.
- [ ] Add a compact prerequisites table (Python, PowerShell, optional Docker, supported OS).
- [ ] Add links to installation, dashboard, CLI, examples, validation, contribution, and security.
- [ ] State clearly which outputs are generated, which reference results are versioned, and
  which directories users should not edit.

## Priority 1 — active directory READMEs

- [ ] Apply one shared structure to active folders: purpose, audience, start here, common
  commands, inputs/outputs, related documentation, and support status.
- [ ] Harmonize `loraflexsim/`, `config/`, `examples/`, `experiments/`, `scripts/`, `results/`,
  `plots/`, `qos_cli/`, and `traffic/` first.
- [ ] Give every PowerShell example a copy-paste-safe Windows 11 form.
- [ ] Distinguish PowerShell from Bash with explicit fenced-code language labels.
- [ ] Replace machine-specific absolute paths with repository-relative paths or placeholders.
- [ ] Standardize vocabulary for dashboard, CLI, scenario, campaign, preset, run, and export.
- [ ] Document whether each folder is an input, generated output, reference artifact, source
  module, or archive.
- [ ] Add bidirectional navigation: each directory README links to the root guide and the root
  documentation map links back to the directory guide.

## Priority 2 — specialized and vendored areas

- [ ] Mark `flora-master/`, `mobile-sfrd_th/`, NumPy/SciPy stubs, and pretest material as
  vendored, compatibility, research, or historical content as appropriate.
- [ ] State whether specialized commands are supported public interfaces or internal tools.
- [ ] Separate reproduction instructions from implementation history.
- [ ] Preserve upstream attribution and licenses while clarifying LoRaFlexSim-specific changes.
- [ ] Add a visible archive banner to every historical README and point readers to the current
  equivalent.

## Priority 3 — quality automation

- [ ] Expand the Markdown audit progressively to every active README without hiding failures
  behind broad allowlists.
- [ ] Check local links, heading anchors, referenced files, and image paths in CI.
- [ ] Add a spelling and terminology allowlist for LoRa/LoRaWAN domain language.
- [ ] Check that Windows paths and commands do not accidentally use Bash-only syntax.
- [ ] Detect stale CLI flags by exercising documented `--help` commands in a smoke test.
- [ ] Validate all YAML/JSON snippets and execute short Python/PowerShell quick starts.
- [ ] Report archive documentation issues separately until the archive reaches the active
  documentation quality level.

## Editorial conventions

- Start with the reader's goal, not the repository's implementation history.
- Prefer short sections, task-oriented headings, and copy-paste-ready commands.
- Explain placeholders immediately and use consistent examples across guides.
- Avoid promises that are not covered by a test, validation report, or cited result.
- Keep active public documentation in English, while clearly labeling retained French research
  notes or translating them as they move into the public surface.
- Use **LoRaFlexSim** for the product, `loraflexsim` for the package/CLI, and code formatting for
  paths, flags, filenames, and commands.

## Definition of done for each README

A README is harmonized when:

1. its status and intended audience are explicit;
2. its product naming and terminology follow this plan;
3. every local link and referenced path exists;
4. every documented command is tested on its stated shell, including Windows 11 where relevant;
5. inputs, outputs, side effects, and generated files are identified;
6. it links to the next useful guide and back to the main documentation surface;
7. it contains no host-specific absolute path, stale entry point, or unexplained legacy name;
8. the documentation consistency check passes.

## Suggested delivery order

1. Root README, installation, dashboard, and CLI guides.
2. Active code, configuration, examples, scripts, and results READMEs.
3. Experiment, plotting, traffic, and QoS documentation.
4. Specialized, vendored, and historical trees.
5. Repository-wide CI enforcement after the active surface is clean.
