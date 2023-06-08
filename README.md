Ansible role: Shipyard
======================

Helm + Helmfile for Docker compose, implemented as an Ansible Role.

## About Shipyard

Shipyard is a tool for orchestrating Docker swarm clusters and stacks from Ansible playbooks.

It is heavily inspired by [Helm](https://helm.sh/) and [helmfile](https://github.com/helmfile/helmfile) and provides the same concepts in a non-kubernetes but swarm-enabled environment. 

## Concepts:

* **Shipyard Chart**: A package defining a docker compose stack and all of it's related files (config files, envs, etc), similar to a Helm Chart, a yum RPM file or a Homebrew formula. A chart contains all resource definitions necessary to deploy and run an application as a Docker Swarm stack, arranged in a specific layout. A chart may be used to deploy a simple application, or a full web application stack with multiple dependencies, such as HTTP servers, database, caches and so on. (similar to a Helm Chart)
* **Shipyard Stack**: an instance of a Shipyard Chart, customized through a `values.yaml` file. (similar to a Helm Release)
* **shipyard.yaml**: a file defining which Charts to instantiate using which values on which docker hosts. (similar to a helmfile.yaml file)

As you can see, the concepts are very similar to Helm and helmfile. The main difference is that Shipyard is not kubernetes-specific and does not require a kubernetes cluster to run. Instead, it uses Docker Swarm to deploy the stacks.

## Prerequisites

The Ansible role assumes that you have pre-configured the target hosts with:

* Docker Swarm (i.e. `docker swarm deploy` works)
* Docker Compose (i.e. `docker-compose` works)
* Docker pull authentication for used images (i.e. `docker pull my-image` works)

## Role variables

* None (see TODO section)

## Usage

### Obtain the role from Ansible galaxy

Setup your `requirements.yml` file to include the role:

```yaml
roles:
  - name: linkorb.shipyard
```

Then run `ansible-galaxy install -r requirements.yml` to install the role.

## Creating a shipyard.yaml file:

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

## Adding the shipyard role to your ansible playbook

In your ansible playbook (usually `site.yml`), add the following:

```yaml
- name: Docker shipyard host configuration
  hosts: my-swarm-hosts # or a group of hosts - expected to be docker swarm managers with docker image pull authentication configured
  tags:
    - shipyard # or any other tag you want to use to run this playbook
  roles:
    - role: linkorb.shipyard # the role from ansible galaxy
```

This will look for the `shipyard.yml` file in the root of the playbook directory, and deploy the stacks defined in there to configured hosts.

## Creating a Shipyard Chart

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

## Target host directory structure

On the target hosts (Docker Swarm managers), the role will create the following directory structure:

```
/opt/shipyard/stacks/
  my-shipyard-stack/
    docker-compose.yml # the rendered docker compose file
    example.conf # the rendered example config file
    # ... etc
```

## Deploying the stacks to Docker Swarm

After the templates are rendered and written to the host, the role will run `docker stack deploy` on the target host to deploy the docker swarm stack.

## Example Shipyard Chart

See the [example/shipyard/chart/whoami](example/shipyard/chart/whoami) directory for an example Shipyard Chart.

## TODO

* [+] Migrate the role on Ansible Galaxy to the `linkorb` organization namespace
* [ ] Make the location of the `shipyard.yaml` configurable
* [ ] Make the location of the Chart directories configurable
* [ ] Support multiple chart directories
* [ ] Make the location of the Stack directories configurable
* [ ] Decide on the use of capitalized `Values`, `Stack` and `Chart` variable names (it follows the Helm convention, but is inconsistent with Ansible's naming conventions)
* [ ] Decide on a packaging mechanism for Shipyard Charts (similar to Helm Charts, yum RPMs, Homebrew formulas, etc). Candidates: tgz, OCI artifacts, git submodules, etc.

## License

MIT. Please refer to the [license file](LICENSE) for details.

## Brought to you by the LinkORB Engineering team

<img src="http://www.linkorb.com/d/meta/tier1/images/linkorbengineering-logo.png" width="200px" /><br />
Check out our other projects at [linkorb.com/engineering](http://www.linkorb.com/engineering).

Btw, we're hiring!

