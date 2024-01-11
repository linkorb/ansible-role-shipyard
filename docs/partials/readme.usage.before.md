## Prerequisites

The Ansible role assumes that you have pre-configured the target hosts with:

* Docker Swarm (i.e. `docker swarm deploy` works)
* Docker Compose (i.e. `docker-compose` works)
* Docker pull authentication for used images (i.e. `docker pull my-image` works)

## Role variables

The role can be customised by some optional variables:

* `shipyard_filename`: Path to your shipyard.yaml file. Default: `{{inventory_path}}`/shipyard.yaml`. This
  file can be a Jinja2 template.
* `shipyard_charts_path`: Path to your charts directory. Default: `{{inventory_path}}`/shipyard/charts`
* `shipyard_stacks_path`: Path to your stacks (values.yaml / values.sops.yaml) directory. Default: `{{inventory_path}}`/shipyard/stacks`
* `shipyard_stacks_docker_secrets`: A list of Docker Secrets.  Default `[]`
* `shipyard_tag`: optionally only deploy stacks with this tag. Default: empty
