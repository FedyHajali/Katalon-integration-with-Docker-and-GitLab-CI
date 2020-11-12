# test-katalon-docker-ci

Sample project of Katalon tests integrated with Gitlab-CI and Docker

## Requirements

Before you begin, ensure:

 1. You have a Gitlab account and you already put your project in a Git repository.
 2. You have Katalon Studio installed and you have created a test suite with one or more test cases.
 3. Docker is installed.
 
 ## Configuration of Gitlab Runner
 
 GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.

 #### Option 1:  

You can choose to [install](https://docs.gitlab.com/runner/install/index.html) the GitLab Runner application on infrastructure that you own or manage. If you do, you should install GitLab Runner on a machine that’s separate from the one that hosts the GitLab instance. 
 
 
 #### Option 2: 
 
 GitLab Runner can also run inside a Docker container or be deployed into a Kubernetees cluster.
 
 We'll see how to run it on a Docker container.
 
 The general rule is that every GitLab Runner command that normally would be executed as: 
 
 

``` bash
gitlab-runner <Runner command and options...>
```

can be executed with: 

``` bash
docker run <chosen docker options...> gitlab/gitlab-runner <Runner command and options...>
```

### Install the Docker image and start the container

To run gitlab-runner inside a Docker container, you need to make sure that the configuration is not lost when the container is restarted.
To do this, there are two options, which are described below. 

#### 1. Use local system volume mounts to start the Runner container: 

``` bash
   docker run -d --name gitlab-runner --restart always \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest
```

#### 2. Use Docker volumes to start the Runner container:

1. Create the Docker volume:

``` bash
docker volume create gitlab-runner-config
```

2. Start the Runner container using the volume we just created: 

``` bash
docker run -d --name gitlab-runner --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v gitlab-runner-config:/etc/gitlab-runner \
    gitlab/gitlab-runner:latest
```

### Registering Runner 

To register a runner using a Docker container: 

1. For local system volume mounts: 

```bash 
docker run --rm -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register

``` 

2. For Docker volume mounts:

```bash 
docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest register
```

Then: 

1. Enter your GitLab instance ```URL```.
2. Enter the ```token``` you obtained to register the runner.
3. Enter a description for the runner.
4. Enter the ```tags``` associated with the runner, separated by commas (e.g. docker, shell, ssh).
5. Provide the ```runner executor```.
6. If you entered docker as your executor, you’ll be asked for the default image to be used for projects that do not define one in .gitlab-ci.yml.
In our case the default image is ```katalonstudio/katalon``` 

For more info: https://docs.gitlab.com/runner/install/docker.html

## Add a configuration file to you repository (.gitlab-ci.yml)
```
image: katalonstudio/katalon
services:
 - docker:dind

stages:
 - test

run_katalon_test_suite:
      stage: test
      tags:
        - docker
      script:
        - katalon -noSplash  -runMode=console -consoleLog -projectPath=
          "<Your_Project_Directory>" -retry=0
          -testSuitePath="Test Suites/<Test_Suite_Name>" -executionProfile=
          "default" -browserType="Chrome (headless)"
```
## Make changes and test

After saving some changes, push them to your git repository. After that, open the repository > CI/CD > Pipelines