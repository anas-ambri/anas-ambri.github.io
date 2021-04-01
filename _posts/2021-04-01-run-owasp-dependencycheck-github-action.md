---
layout: post
title: Run OWASP DependencyCheck as a GitHub action on a gradle project
comments: true
categories: blog
published: true
disqus: y
tags:
 - 
---

Say, you are a very security-oriented developer, who doesn't want to allow [any publicly disclosed vulnerabilities](https://www.forbes.com/sites/forbestechcouncil/2020/12/04/is-a-data-breach-lurking-in-your-software-supply-chain/) in their project dependency tree. To that end, you have [OWASP's DependencyCheck](https://owasp.org/www-project-dependency-check/) configured on your gradle project as a [plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/index.html). For example, such a configuration could look like the following:

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.owasp:dependency-check-gradle:6.1.5'
    }
}

apply plugin: 'org.owasp.dependencycheck'

dependencyCheck {
    failBuildOnCVSS = 2
    cveValidForHours=24
}
```

This way, every time you run the command `./gradlew check` (or `./gradlew dependencyCheckAnalyze` for that matter), the build pipeline automatically checks NIST's CVEs database for any vulnerabilities against the dependencies of your project, and fails the local build if a CVE is found with a CVSS score higher than or equal to 2.

It turns out, however, that not everyone on your development team is as diligent as you are. Yes, you could add a git hook to your repository to force the `check` task to execute on each PR. But that can easily be skipped by passing `--no-verify` to the `git push` command.

## GitHub Action

If your project is hosted on GitHub, a very simple solution is to just run the check task as a GitHub Action. Just drop a `.yml` file under `.github/workflows/`, with the following content:

```yml
name: DependencyCheck with Gradle

on: pull_request

jobs:
  depCheck:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - uses: eskatos/gradle-command-action@v1
        with:
          arguments: dependencyCheckAnalyze
```

And that's it! This simple workflow will execute on each PR, and report the result as a notification in GitHub, assuming you have configured it to do so in your [settings](https://github.com/settings/notifications).

## Optimizing the workflow

As part of its operation, the plugin will automatically download a copy of the NIST NVD database, and cache it locally. While this is ideal for local runs, it doesn't work for a CI setup. Basically, we need to introduce a way to cache the database, which normally lives under `~/.gradle/dependency-check-data`. The solution is to add the following step before running the gradle command action:

```yml
      - name: Cache Dependency Check DB
        uses: actions/cache@v2
        with:
          path: ~/.gradle/dependency-check-data
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          restore-keys: ${{ runner.os }}-gradle
```

## Publishing the result

The one remaining thing is to make sure the results of the dependency check are published, no matter the result. This is done using a simple upload artifact action, after the check task completes:

```yml
      - name: Backup Report
        if: "${{ always() }}"
        uses: actions/upload-artifact@v2
        with:
          name: dependency-check-report.html
          path: ./build/reports/dependency-check-report.html
```

## Putting it altogether

And there you go: a dependency check on any PR, running alongside your repo.

<div class="img-center"><img src="/images/DependencyCheckGitHubAction/dependency_check_github_action.png"/> </div>