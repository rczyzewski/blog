---
title: 'Publishing artifacts to maven central via GitHub actions'
subtitle: "Migration from OSSRH to Central portal"
author:
  - rczmdp
categories: [ GitHub actions]
tags: [ central portal, deployment, GitHub ]
description: |
  In recent months, OSSH has been decomisioned, and it caused errors in my GitHub release pipelines.
---

## Unable to deploy

In recent months, OSSRH Account Managment Portal has been decomisioned.
As a result, I have lost access
to [s01.oss.sonatype.org/](https://s01.oss.sonatype.org/#nexus-search;quick~io.github.rczyzewski)
Also my GitHub pipelines stoped to work.
With the new service responsible for managing accounts, came also a new maven plugin for making releases, the deploymnet
pipeline needs to be updated.

Error trying to use the old deploy method looked like:

```log
Error:  
  Failed to execute goal org.sonatype.plugins:nexus-staging-maven-plugin:1.6.13:
    deploy (injected-nexus-deploy) on project tortilla: 
  Execution injected-nexus-deploy of goal org.sonatype.plugins:nexus-staging-maven-plugin:1.6.13:deploy failed: 
  Nexus connection problem to URL [https://s01.oss.sonatype.org/ ]: 401 - Unauthorized -> [Help 1]
```

Luckily, I have found this page: [Central Repository FAQ](https://central.sonatype.org/faq/ossrh-account-management/)
where I read:

> Our previous OSSRH Account Management Portal has been decomissioned. The OSSRH account profiles were merged with the
> same authentication provider used by Central Portal.

## Setting up a new account with migrated namespaces.

I have created a new account, using email and password method, in the 'Central Portal'.
Then I send an email, where I asked to migrate the namespace from OSSRH to CentralPortal.
As a response I get:

> Hello Rafal,
>
> Your namespace is registered in OSSRH and would require migration to Central Portal.
>
> While it is possible for us to migrate you to Central Portal, please note publishing to Central Portal and to
> Central's legacy NXRM (also called OSSRH) are two different methods: they use different connection endpoints, plugins
> and authorization credentials. Please review our documentation to find out the details of each method.
>
>In addition to that, please note that SNAPSHOT versions should not be used to sync with Maven Central (for more detail
> on this please review our FAQ). If you wish for your artifacts to be available from Maven Central, you will have to
> publish a release version. Snapshots are supported in legacy OSSRH.
>
>Publishing through Gradle is also not yet officially supported in Central Portal, as Portal is still in early-access.
> There are however several third-party plugins for Gradle that might be of interest if you wish to use Gradle. Publishing
> through Maven via our Central Publishing Maven Plugin is fully supported.
>
>If you are interested to publish from Central Portal, we must first revoke your deployment access from OSSRH before you
> can verify your namespace at https://central.sonatype.com/.
>
>Please let us know if you have any questions.
>
>Thank you,
> The Central Team

## Different way of deploying:

In central portal documentation I have noticed that the way of deploying via maven has been changed.
The new configuration in pom.xml should look like:

```xml

<build>
  <plugins>
    <plugin>
      <groupId>org.sonatype.central</groupId>
      <artifactId>central-publishing-maven-plugin</artifactId>
      <version>0.4.0</version>
      <extensions>true</extensions>
      <configuration>
        <publishingServerId>central</publishingServerId>
        <tokenAuth>true</tokenAuth>
      </configuration>
    </plugin>
  </plugins>
</build>
```

Where `publishinServerId` refers to the entry in `m2/settings.xml` like follows:

```xml

<server>
  <id>${server}</id>
  <username>*******</username>
  <password>********************************************</password>
</server>
```

The real values were obtained from the central portal.
Generating a new username and token for the user:
https://central.sonatype.com/account
![generating](/assets/posts/central_portal_token_generation.png)

## Keeping tokens secret among the secrets
But there was a small problem, to apply it immediatly, I don't wanted to have a token stored in repository,
where everyone can see it. So solution seems to be using secrets, and in `.m2/settings.xml` use references to
environment variables

```xml

<server>
  <id>central</id>
  <username>${env.MAVEN_USERNAME}</username>
  <password>${env.MAVEN_PASSWORD}</password>
</server>
```

GitHub secretes has been set up by using project settings page: ![picture](/assets/posts/github_secretes.png)

The remaining part was to connect environment variables with secrets.
This was done by the GhitHub actions: [setup-java](https://github.com/actions/setup-java)

```yaml
name: release and push to central
on:
  push:
    tags:
      - '*'
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Java for publishing to Maven Central Repository
        uses: actions/setup-java@v1
        with:
          java-version: 8
          server-id: central
          server-username: MAVEN_USERNAME
          server-password: MAVEN_PASSWORD
          gpg-private-key: ${{ secrets.OSSRH_GPG_SECRET_KEY }}
          gpg-passphrase: MAVEN_GPG_PASSPHRASE
      - name: Publish to the Maven Central Repository
        run: |
          mvn \
          --no-transfer-progress \
          --batch-mode \
          -pl guacamole-core,guacamole-dockertest,guacamole-om  \
          -P release \
          clean package deploy
        env:
          MAVEN_USERNAME: ${{ secrets.CENTRAL_USERNAME }}
          MAVEN_PASSWORD: ${{ secrets.CENTRAL_PASSWORD }}
          MAVEN_GPG_PASSPHRASE: ${{ secrets.MAVEN_GPG_PASSPHRASE }}
```

The above procedure uses two steps: one is to just perform checkout from the repository, the second one prepares java
environment.


Finally, generated `.m2/settings.xml` looked like:

```xml

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>central</id>
      <username>${env.MAVEN_USERNAME}</username>
      <password>${env.MAVEN_PASSWORD}</password>
    </server>
    <server>
      <id>gpg.passphrase</id>
      <passphrase>${env.MAVEN_GPG_PASSPHRASE}</passphrase>
    </server>
  </servers>
</settings>
```
The server `gpg.passphrase` is required for a different gpg maven plugin.
https://maven.apache.org/plugins/maven-gpg-plugin/usage.html Signing the artifacts is required to be able to publish,
but is irrelevant to this article.

## More strict verification of projects
And it was almost it, almost, because there were new exciting errors related to lack of required project information:

> pkg:maven/io.github.rczyzewski/guacamole-core@0.1.1-RC3:
> - Developers information is missing
> - License information is missing
> - Project URL is not defined
> - Project description is missing
> - SCM URL is not defined

That was the error from my side: all published components had a common parent, and that parent has those thing setups -
but the parent itself hasn't been published, so I deployed it too.

## Updating versions of GitHub actions

As the latest changes has been performed about a year ago, there where a new version of steps available.
So  `actions/checkout@v2` and  `actions/setup-java@v1` were both updated to the latest version.
`actions/setup-java` required specyfing java-version in a new style( 1.8 -> 8) and also specyfing the distribution of
the java.

```yaml
distribution: 'zulu'
java-version: 8
```

In this way, my two babies: 
* [Guacamole](https://github.com/rczyzewski/guacamole)
* [Tortilla](https://github.com/rczyzewski/tortilla)
 has both updated and working release pipelines.



