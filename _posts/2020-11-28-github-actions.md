---
title: Github Actions
category: devops
tags:
- Spring Boot
- devops
- deployment
- Github actions
excerpt: "A post on creating Github actions to build a spring-boot application automatically. The common and popular github actions are also discussed."
---

Continuous Integration(CI) & Continuous Delivery(CD) are one of the necessary steps in the development of a software. In simple terms, Continuous Integration is validating, testing and integrating the code changes made by a developer. Continuous delivery means releasing/deploying the software in an automated manner. CI & CD saves developer time, that is usually spent in manual testing, enforcing the code format etc. It also helps in reducing the release cycle time.

Github Actions helps developers automate the CI & CD process. Its similar to Jenkins or CircleCi or Travis Ci. Let us see in this post, how Github actions simplifies CI/CD for you.

### Sample Github action
To get started all is needed is a yaml file in the `.github/workflows` folder.  Below is the sample Github action yaml

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml

```

As you see, the yaml has a list of steps, each one being a Github action. 
1. The action `checkout` clones the code from repository to the Github runner and uses the action with tag v2
2. The  action `setup-java` sets up the java environment in the runner by installing jdk. Note the argument `java-version` given to the action that specifies the JDK version to install in the runner. 
3. The action runs mvn to build the java application. You might be wondering how we can run mvn command without installing maven. The Github hosted runners i.e the default runners provided by Github, comes installed with a predefined set of binaries. You can find the Github runner and the specifications for each runner [here][github runner pre-installed softwares].

### Common github actions
 There are many official and custom actions created by developers which will be crucial to get your workflow up and running. Below are the set of actions, I have used personally or seen widely used in open source projects and they are really great.
 - `actions/cache` - This helps cache dependencies. For example, maven local repository (.m2) directory can be cached to reduce the build time of your workflow
 - `Setup actions` i.e actions to setup java, python, node, ruby, Haskell, dotnet, Go etc. These actions helps you set the build environment for the programming languages. Tomorrow, we want to use a different version and in the workflow file, we can simply modify the version and we are good to go
 - `Code quality` actions like [these][code quality actions].
 - `Docker actions` to build & push docker images. It is nicely explained in this [page][docker build push action].
 
 For a curated list of the actions, you can checkout this [repository][github actions curated].

### Benefits
- As a developer, I don't need to bother about having infra to build the repository. Github takes care of the infra. Also, Github offers enough free build minutes, which should be sufficient for smaller teams or individuals. Also there is provision that we can create & host your own runners to run the workflow. More details [here][self-hosted runners].
- There are so many open source Github actions ready to use. If needed, you can create one for yourself. Details on creating your own Github action [here][custom github action].
- The capability to run the workflow against various [events][workflow trigger] such as pull requests, branch updates, releases etc
- Viewing the results of each Github workflow as part of the pull request, making the code review process much simpler.
- Having more than one workflow for a repository

[self-hosted runners]: https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners/about-self-hosted-runners
[custom github action]: https://docs.github.com/en/free-pro-team@latest/actions/creating-actions
[workflow trigger]: https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows
[github runner pre-installed softwares]: https://docs.github.com/en/free-pro-team@latest/actions/reference/specifications-for-github-hosted-runners
[github actions curated]: https://github.com/sdras/awesome-actions
[code quality actions]: https://github.com/marketplace/category/code-quality
[docker build push action]: https://www.docker.com/blog/docker-github-actions/
