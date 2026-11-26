---
name: opentofu-preferences
description: Applies tested conventions for OpenTofu/Terraform HCL - conditional resources with lifecycle enabled, variables.tf layout and validation error messages, output ordering, child module structure and commit-pinned community modules. Use when writing, reviewing or refactoring .tf files or tofu modules, even if the user says "Terraform". Not for state backends, CI pipelines or testing strategy.
license: Apache-2.0
compatibility: Requires OpenTofu >= 1.11 (lifecycle enabled meta-argument).
---

# OpenTofu Preferences

Preferred conventions for OpenTofu code. They override generic Terraform guidance, including the `terraform-skill` skill if installed.

## Routing

| Task                                                | Read                                                               |
| --------------------------------------------------- | ------------------------------------------------------------------ |
| Add or edit module variables, validation, defaults  | [references/variables.md](references/variables.md)                 |
| Add a module, choose a community module, wire roots | [references/project-structure.md](references/project-structure.md) |
| Conditional resources, outputs, section headers     | Below                                                              |

## Conditional resources

Use `lifecycle { enabled }` for a resource or module that may or may not exist. It removes the `[0]` index that `count` forces on every reference.

```hcl
resource "aws_route53_record" "this" {
  zone_id = var.zone_id
  name    = "blog.example.dev"
  type    = "A"
  # ...

  lifecycle {
    enabled = var.create_dns
  }
}

output "dns_record_fqdn" {
  value = aws_route53_record.this != null ? aws_route53_record.this.fqdn : null
}
```

- A disabled resource evaluates to `null`. Guard references with `!= null ? ... : null`; avoid `try()` because it hides unrelated errors.
- Use boolean variables (`var.enabled`, `var.create_dns`), set per environment, to control what differs between environments. Do not compare the environment name (`var.environment == "prod"`): it hides why the behaviour differs and breaks when a new environment is added.
- When refactoring `count = cond ? 1 : 0`, convert it to `enabled` and drop the `[0]` from references. OpenTofu moves the state from `x[0]` to `x` automatically, so no `moved` block is needed.
- Where `count` must stay (e.g. `enabled` is rejected), reference it with `one(x.this[*].attr)` instead of `x.this[0].attr`. `one()` returns `null` for zero instances, whereas `[0]` fails.

## Input validation

Use `validation` blocks on variables, not `check` blocks. `check` failures are warnings and do not stop an apply. Error message formats are in [references/variables.md](references/variables.md).

## Outputs

Alphabetise outputs so diffs stay minimal and outputs are easy to find.

## Section headers

Separate sections in any `.tf` file with this banner (120 characters wide, sentence-case title):

```hcl
# ======================================================================================================================
# Required values
# ======================================================================================================================
```

## Gotchas

- `lifecycle { enabled }` exists only in OpenTofu 1.11+ and fails under Terraform. It cannot be combined with `count`/`for_each`, and its value must be a known, non-sensitive, non-null boolean at plan time.
- Registry module `version` cannot take a commit hash. To pin a community module by commit, use a `git::` source with `?ref=<sha>` (see [references/project-structure.md](references/project-structure.md)).
- Registry lookups (MCP server or registry website) return release tags, not commits. Resolve the tag with `git ls-remote https://github.com/<org>/<repo> 'refs/tags/<tag>*'`; if a `^{}` line appears, pin that SHA instead.
- Use the `tofu` CLI, not `terraform`.

## Validate before finishing

1. `tofu fmt -recursive`
2. In each affected root module (including roots that call a child module you changed): `tofu init -backend=false` then `tofu validate`. Validate needs providers and modules installed; skipping the backend avoids needing state credentials.
3. If you converted `count` to `enabled`, run `tofu plan` and confirm the plan shows moves rather than destroy/create. This needs a full `tofu init` with backend access; if that's unavailable, ask the user to run it.
4. Fix any failures and repeat.
