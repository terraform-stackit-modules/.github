# Terraform STACKIT Modules

Open-source Terraform modules for the [STACKIT](https://www.stackit.de/) cloud —
a community library of small, composable, single-service modules, published to the
[Terraform Registry](https://registry.terraform.io/namespaces/terraform-stackit-modules) as well as [Open Tofu Registry](https://search.opentofu.org/modules)

Inspired by [terraform-aws-modules](https://github.com/terraform-aws-modules) and the AWS Architecture & Infrastructure conventions:
**one module per service**, thin and predictable.

## Design principles

- **One service, one module** — no mega-modules. A security group is its own repo, not a flag on the network module.
- **Composable** — modules consume each other through the public registry (e.g. an example wires the published `network` module + a NIC + a server).
- **Self-contained examples** — every `examples/basic` runs with `project_id` alone, so it is testable end to end.
- **Tested** — Terratest in CI against a real STACKIT project, plus `fmt` / `validate` / `tflint` / `terraform-docs` via pre-commit.
- **SemVer releases** — automated with semantic-release; each tag is ingested by the registry.

## Contributing

Each module lives in its own repository, issues and pull requests are welcome.

## License

Modules are released under the [MIT License](https://opensource.org/licenses/MIT).
