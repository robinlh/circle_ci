# Build Singularity image and push to Hinkskalle registry with CirclCI

This is a proof of concept repository with a basic Singularity container recipe and accompanying files that uses a CirclCI pipeline to build a Singularity image from the recipe (with "latest" tag) and push it to a hosted Hinkskalle container registry server. Building the Singularity image is done with a [custom Docker image](https://github.com/robinlh/docker-singularity) with Singularity installed in it that gets pulled from Dockerhub. It would be entirely possible to install Singularity on the CirclCI VM that gets spun up for a build, but having Singularity pre-installed in a Docker container vastly speeds up the build.

In order to trigger a new build:
- clone repositroy
- make change to Singularity recipe
- commit and push change

## CircelCI config structure

### Jobs
In this case we've just got one job with multiple steps, but in general each job is given a name and put under the jobs section:
```
jobs:
   build-push:
   ...
   job-2: (if we had another job, perhaps tests in future)
```
We're building on a Linux virtual machine, specified by the `machine` excecutor block that goes under the job name (uses Ubuntu 20):
```
jobs:
  build-push:
    machine:
      image: ubuntu-2004:202010-01
```
Next we define the steps in the job. These steps can all be written in the same block and piped to a single `- run` but it makes it easier to understand what's happening if we break the steps up into their own commands. Important is the `- checkout` step which pulls the contents of the repository to a CirclCI server:
```
jobs:
  build-push:
    machine:
      image: ubuntu-2004:202010-01
    steps:
      - checkout
      - run:
          name: "Name of step that will show up on CirclCI site"
          command: echo "this is a command"
```
### Workflows
The workflows block is where we string together alll the jobs for a pipeline and define the context CirclCI must use to reference environment variables. Again in this case the workflow just runs the one job I've defined:
```
workflows:
  build-push-singularity:
    jobs:
      - build-push:
          context: dockerhub
```
