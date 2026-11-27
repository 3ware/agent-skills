# Variables

## Layout of `variables.tf`

| Section           | Contents                                                  |
| ----------------- | --------------------------------------------------------- |
| Validation locals | Locals reused by validation rules; at the top of the file |
| Required values   | Variables without defaults, alphabetised                  |
| Optional values   | Variables with defaults, alphabetised                     |

```hcl
# ======================================================================================================================
# Validation locals
# ======================================================================================================================
locals {
  supported_regions = ["eu-west-1", "eu-west-2"]
}

# ======================================================================================================================
# Required values
# ======================================================================================================================
variable "name" {
  description = "Name prefix for all resources"
  type        = string
}

# ======================================================================================================================
# Optional values
# ======================================================================================================================
variable "allowed_regions" {
  description = "Regions the module may deploy into"
  type        = list(string)
  default     = ["eu-west-2"]
  nullable    = false
}
```

## Design

- ✅ Group related values in an object (e.g. a `vpc_config` object for VPC settings) to keep the module interface small.
- ✅ Give variables consumed by `for` or `for_each` a default and `nullable = false`, because iterating over `null` fails.
- ✅ Put values repeated across validation rules in locals under Validation locals so they have a single source of truth.

## Descriptions

- ✅ Clear and concise.
- ❌ Never start with "The", "A", "Whether", "Optional" or "Required".

| ❌                   | ✅               |
| -------------------- | ---------------- |
| `The VPC CIDR block` | `VPC CIDR block` |

## Validation error messages

Primitive and structural types expose the invalid value natively; collection types do not.

| Variable type | `error_message`                                                    |
| ------------- | ------------------------------------------------------------------ |
| Primitive     | String documenting the acceptable values                           |
| Structural    | String documenting the acceptable values                           |
| Collection    | `format(...)` exposing the invalid input(s) and the allowed values |

```hcl
# Primitive / structural
error_message = "Region must be one of: eu-west-1, us-east-1."

# Collection
error_message = format(
  "Invalid region(s) specified. Received: %v. Required: %v.",
  var.allowed_regions,
  join(", ", local.supported_regions)
)
```
