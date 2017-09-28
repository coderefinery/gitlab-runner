# GitLab runner Ansible role

Configure a GitLab runner using Ansible. Supports containerized and rpm-based 
installation of the multirunner. Allows flexible configuration during registration 
via environment variables without any hardcoded registration options.

## Variables

  * Mandatory
    * `runner_configs`: a list of environment variable dictionaries - see below
      for an example
    * `gitlab_server_fqdn`: the FQDN of the GitLab server that the runner
      registers to

  * Optional
    * `gitlab_runner_tag`: what version of the GitLab runner image to use
    * `self_signed_certificate`: self-signed cert to use for registering to
      GitLab (if your GitLab installation has a cert signed by a recognized CA,
      you don't need to set this)
    * `initial_shared_runners_registration_token`: the token used for shared
      runners (you don't need to set this if you don't plan to have shared
      runners)
    * `runner_deployment_type`: whether to deploy the runners by creating a
      "gitlab-runner" Docker container per runner configuration (option 'docker')
      or by installing a native binary from GitLab's yum repository (option 'rpm').
      The default is 'docker'.
    * `runner_concurrency`: optionally set global 'concurrent' -parameter
      in the gitlab-runner config file.

## Runner configuration

The `runner_configs` list contains configuration using configuration variables
for one or more runners. Here is an example for configuring several runners:

```
runner_configs:
  - RUNNER_NAME: "centos-dedicated"
    DOCKER_IMAGE: "centos"
    REGISTER_LOCKED: "true"
    state: "started"
  - RUNNER_NAME: "ubuntu-dedicated"
    DOCKER_IMAGE: "ubuntu"
    REGISTER_LOCKED: "true"
    REGISTRATION_TOKEN: "{{ vaulted_ubuntu_dedicated_reg_token }}"
    state: "started"
  - RUNNER_NAME: "ubuntu-shared"
    DOCKER_IMAGE: "ubuntu"
    REGISTRATION_TOKEN: "{{ initial_shared_runners_registration_token }}"
    state: "started"
  - RUNNER_NAME: "alpine-shared"
    DOCKER_IMAGE: "alpine"
    REGISTRATION_TOKEN: "{{ initial_shared_runners_registration_token }}"
    state: "started"
```

In this example, two dedicated and two shared runners are created. One of the
dedicated runners already has a registration token available
("vaulted_ubuntu_dedicated_reg_token"), while the other one is added without
setting a registration token. This means that it won't be registered initially,
but will be created without registration. The registration token can then be
added later on to register the runner.

The shared runners use the token for shared runners
("initial_shared_runners_registration_token") as their registration token. One
of them uses a Ubuntu base image while the other uses an Alpine Linux base
image.

You can get a full list of all the available environment variable config
options by running `gitlab-ci-multi-runner help register` on a container or a
server that has gitlab-ci-multi-runner installed.

In addition to environment variables, you can set a state for each runner. This
can be whatever the state for an Ansible docker_container can be.
