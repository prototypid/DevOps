# Artifact repository with Nexus

What is Artifact repository?
Artifacts are application build into single portable, sharable file. It can have different formats - Jar, War, Zip, Tar based on what tools or programming language you use.

Artifact repository is where you store those files. Repository for each file/artifact type, there are different artifact formats.

Artifact repository manager is a software that manages different types of artifacts.

Artifact repository manager allows us to store or upload different built artifacts and retrieve artifacts later. Nexus.
There are also public repository manager mvnrepository for Java and npm for JS.

## Installing Nexus

1. `sudo apt install openjdk-8-jdk`
2. `cd /opt`
3. `wget https://download.sonatype.com/nexus/3/latest-unix-tar.gz`
4. untar file - `tar -zxvf latest-unix-tar.gz` It creates two folder - nexus-3.28 sonatype-work, nexus folder contains runtime and application  of nexus, sonatype-work folder contains own config for Nexus and data. sonatype-work folder also contains subdirectories (plugins) depending on your nexus configuration, IP addresses that accessed Nexus, logs of Nexus app, Your uploaded files and metadata. This folder can be used to backup nexus data.
5. Starting nexus service, but it is not a good practice to start a service as a root user. Create own user for service (eg. Nexus). 
6. `adduser nexus`
7. change permission for the directories: `sudo chown -R nexus:nexus sonatype-work nexus-3.28`
8. change nexus configuration to run nexus as nexus user: `sudo vim nexus-3.54.1-01/bin/nexus.rc` change `run_as_user='nexus'`
9. `su - nexus`
10. `/opt/nexus-3.54.1-01/bin/nexus start`
11. to check nexus is running: `ps aux | grep nexus` && `netstat -lnpt`
12. Now this is accessible at port 8081.
13. Get default password `cat /opt/sonatype-work/nexus3/admin.password` username: admin
 

## Repository types in NExus

We can have different repository of different formats such as helm charts, docker images etc.

Repository can be of 3 types:

- hosted
- group
- proxy

**Proxy Repository**:

This is a repository that is linked to remote repository. If a component is required from the remote repository, it will go through the proxy instead of directly to the remote and check if component is available locally and if not available then the request will be forwarded to the remote repository and the component will be retrieved from the remote and stored locally in nexus. Nexus will act as a cache.

**Hosted Repository**:

This is a repository for the primary storage for artifacts and components e.g.  an organization repo.

**Group Repository**:

It is a powerful feature of Nexus repository manager because you may have multiple individual repository in Nexus and each one has its own purpose, however if you need to use multiple repositories in your application. So you don't want to configure different endpoints for each repository. So this repository allows you to combine multiple repository even groups in single repository, so you can use single endpoint for their application which will give them access to multiple repositories.

## Publishing artifacts to repository

VID - 5

Create a new user for developer.

## Nexus REST API

Query Nexus Repository for different information such as which component, which repo, what are the versions. This information is needed in you CI/CD Pipeline.

Get all repositories: `curl -u user:pwd -X GET 'http:/[url]:[port]/service/rest/v1/repositories'`

all components in a repo: `curl -u user:pwd -X GET 'http:/[url]:[port]/service/rest/v1/components?repository=[repo-name]'`

## Blob Store

Nexus used **Blob store** as a storage to store its components. Blob store is an internal storage mechanism for binary parts of artifacts. Blob store can be on the local storage or cloud storage. Each blob store can be used by one or multiple repositories.

VID - 7

