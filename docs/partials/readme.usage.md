## Usage

### Obtain the role from Ansible galaxy

Setup your `requirements.yml` file to include the role:

```yaml
roles:
  - name: linkorb.shipyard
```

Then run `ansible-galaxy install -r requirements.yml` to install the role.

### Creating a shipyard.yaml file:

The `shipyard.yaml` file defines which stacks get deployed to which hosts. It is similar to a helmfile.yaml file.

```yaml
# shipyard.yaml
stacks:
  - name: my-traefik
    chart: traefik
    host: swarm-host-a
    values: my-traefik/values.yaml

  - name: my-whoami
    chart: whoami
    host: swarm-host-a
    values: my-whoami/values.yaml

  - name: my-mariadb
    chart: mariadb
    host: swarm-host-a
    values: my-mariadb/values.yaml

  - name: my-whoami-b
    chart: whoami
    host: swarm-host-b
    values: my-whoami-b/values.yaml
```

### Adding the shipyard role to your ansible playbook

In your ansible playbook (usually `site.yml`), add the following:

```yaml
- name: Docker shipyard host configuration
  hosts: my-swarm-hosts # or a group of hosts - expected to be docker swarm managers with docker image pull authentication configured
  tags:
    - shipyard # or any other tag you want to use to run this playbook
  roles:
    - role: linkorb.shipyard # the role from ansible galaxy
      vars:
        shipyard_filename: "shipyard/shipyard.yaml"
        shipyard_charts_path: "shipyard/charts"
        shipyard_stacks_path: "shipyard/stacks"
```

The role accepts 3 (optional) configuration variables:

* `shipyard_filename`: Path to your shipyard.yaml file. Default: `{{inventory_path}}`/shipyard.yaml`
* `shipyard_charts_path`: Path to your charts directory. Default: `{{inventory_path}}`/shipyard/charts`
* `shipyard_stacks_path`: Path to your stacks (values.yaml / values.sops.yaml) directory. Default: `{{inventory_path}}`/shipyard/stacks`

This will look for the `shipyard.yml` file in the root of the playbook directory, and deploy the stacks defined in there to configured hosts.

### Creating a Shipyard Chart

Directory structure of a Shipyard Chart:

```
my-shipyard-chart/
  Chart.yaml # the chart metadata
  LICENSE # the license for this chart
  README.md # the readme for this chart
  values.yaml # the default values for this chart
  templates/ # the jinja2 templates for this chart
    docker-compose.yml # the docker compose template file for this chart
    example.conf # an example config file template for this chart
    env.example # another example config file template for this chart
```

The Shipyard role will copy over all files in the `templates/` directory onto the target host, and then render them using the values from the `values.yaml` file.

### values.yaml / values.sops.yaml and chart default values

Every stack (one instance of a chart), takes a values file containing the values for that instance of the chart.
The values are loaded from `{{shipyard_stacks_path}}/{{stack_name}}/values.yaml`. If a `values.sops.yaml` is detected, it is also loaded and decrypted automatically (based on the `.sops.yaml` in the root of your repo).

Every chaty provides a default values.yaml too. Any stack-level value that remains undefined will be set to the chart's default value.

The loading (and override precedence) order is:

1. the default values from the chart
2. the values.yaml from the stack
3. the values.sops.yaml from the stack

## Target host directory structure

On the target hosts (Docker Swarm managers), the role will create the following directory structure:

```
/opt/shipyard/stacks/
  my-shipyard-stack/
    docker-compose.yml # the rendered docker compose file
    example.conf # the rendered example config file
    # ... etc
```

### Deploying the stacks to Docker Swarm

After the templates are rendered and written to the host, the role will run `docker stack deploy` on the target host to deploy the docker swarm stack.

### Example Shipyard Chart

See the [example/shipyard/chart/whoami](example/shipyard/chart/whoami) directory for an example Shipyard Chart.


