# Agent Skills

[Agent Skills](https://agentskills.io) encoding 3ware conventions, loadable by any skills-compatible agent.

[![GitHub release](https://img.shields.io/github/release/3ware/agent-skills?include_prereleases=&sort=semver&color=green)](https://github.com/3ware/agent-skills/releases/) [![issues - workflows](https://img.shields.io/github/issues/3ware/agent-skills)](https://github.com/3ware/agent-skills/issues) [![CI](https://img.shields.io/github/actions/workflow/status/3ware/agent-skills/pr-check.yaml?label=CI&logo=githubactions&logoColor=white)](https://github.com/3ware/agent-skills/actions/workflows/pr-check.yaml)

| Skill                                                           | Purpose                                                                       |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [opentofu-preferences](../skills/opentofu-preferences/SKILL.md) | OpenTofu coding standards: conditional resources, variables, modules, outputs |

## Configuration

| File                                                          | Purpose                                                                    |
| ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [mise.toml](../mise.toml)                                     | Pins `uv`; its `postinstall` hook installs `skills-ref` at a pinned commit |
| [.trunk/trunk.yaml](../.trunk/trunk.yaml)                     | Defines the `skills-ref` linter, run on every `SKILL.md`                   |
| [.trunk/setup-ci/action.yaml](../.trunk/setup-ci/action.yaml) | Installs mise tools in CI before Trunk runs                                |

## Requirements

- [mise](https://mise.jdx.dev)
- [Trunk](https://marketplace.visualstudio.com/items?itemName=Trunk.io)

## Usage

Add `apm.yaml` to the root of your repository and run `apm install` ([APM](https://github.com/microsoft/apm))

> [!TIP]
> Use the OpenTofu MCP server and extension rather than HashiCorp's because `terraform-ls` and the Terraform MCP server may report OpenTofu-only features, such as `lifecycle { enabled }`, as errors.

```yaml
name: my-infra
version: 1.0.0
targets: [copilot]
dependencies:
  apm:
    - antonbabenko/terraform-skill/skills/terraform-skill#<tag>
    - git: 3ware/agent-skills
      ref: <tag>
      skills: [opentofu-preferences]
  mcp:
    - name: opentofu
      registry: false
      transport: http
      url: https://mcp.opentofu.org/mcp
```

This installs `opentofu-preferences` alongside [terraform-skill](https://github.com/antonbabenko/terraform-skill) (testing, state, CI), with the [OpenTofu MCP server](https://github.com/opentofu/opentofu-mcp-server) for registry lookups. For code intelligence, install the [OpenTofu VS Code extension](https://marketplace.visualstudio.com/items?itemName=opentofu.vscode-opentofu), which bundles the [tofu-ls](https://github.com/opentofu/tofu-ls) language server.

## Maintenance

Install tools and validate:

```sh
mise install
trunk check
```

> [!IMPORTANT]
> The `postinstall` hook only runs when `uv` is installed. After changing the `skills-ref` commit in `mise.toml`, run `mise install --force uv`.
