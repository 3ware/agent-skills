# Project Structure

## Layout

| Path                       | Purpose                                                       |
| -------------------------- | ------------------------------------------------------------- |
| `iac/environments/<name>/` | Root module per account; composes child modules               |
| `iac/modules/<name>/`      | Child module with a single function (`vpc`, `bastion`, `rds`) |
| `iac/modules/metadata/`    | Standardises resource naming and tagging                      |

## Choosing a source for a resource

| Situation                                             | Use                                           |
| ----------------------------------------------------- | --------------------------------------------- |
| Well-maintained `terraform-aws-modules` module exists | The community module, pinned by commit hash   |
| No suitable community module                          | Plain resources inside a focused child module |

Root modules only compose child modules. Keep each child module to a single function so it can be reused and reviewed in isolation.

## Rules

- ✅ Include the metadata module in each child module so names and tags stay consistent across accounts.
- ✅ Pin community modules to a full commit SHA. Branches and tags can be moved, so a commit is the only immutable reference (supply-chain safety).
- ❌ Do not pin to branches or tags, and do not use the registry `version` argument (it cannot take a SHA).

```hcl
module "vpc" {
  source = "git::https://github.com/terraform-aws-modules/terraform-aws-vpc.git?ref=277e8947b1267290988e47882d8dc116850929be"
  # ...
}
```
