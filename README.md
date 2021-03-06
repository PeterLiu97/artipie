<img src="https://www.artipie.com/logo.svg" width="64px" height="64px"/>

[![EO principles respected here](https://www.elegantobjects.org/badge.svg)](https://www.elegantobjects.org)
[![DevOps By Rultor.com](http://www.rultor.com/b/yegor256/artipie)](http://www.rultor.com/p/yegor256/artipie)
[![We recommend IntelliJ IDEA](https://www.elegantobjects.org/intellij-idea.svg)](https://www.jetbrains.com/idea/)

[![Build Status](https://img.shields.io/travis/yegor256/artipie/master.svg)](https://travis-ci.org/yegor256/artipie)
[![Javadoc](http://www.javadoc.io/badge/com.yegor256/artipie.svg)](http://www.javadoc.io/doc/com.yegor256/artipie)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/yegor256/artipie/blob/master/LICENSE.txt)
[![Hits-of-Code](https://hitsofcode.com/github/yegor256/artipie)](https://hitsofcode.com/view/github/yegor256/artipie)
[![Maven Central](https://img.shields.io/maven-central/v/com.yegor256/artipie.svg)](https://maven-badges.herokuapp.com/maven-central/com.yegor256/artipie)
[![PDD status](http://www.0pdd.com/svg?name=yegor256/artipie)](http://www.0pdd.com/p?name=yegor256/artipie)

This is a simple experimental binary artifacts manager.

## How does it work

Artipie uses external server implementation to start itself,
one of possible servers is https://github.com/artipie/vertx-server/

Server accepts `Slice` interface and serves all requests to encapsulated `Slice`.
Server can be started with a single adapter module (since all adapters implements `Slice` interface)
or with Artipie assembly.

Artipie reads server settings from yaml file and contructs
`Storage` to read repositories configurations.

On each request it reads repository name from request URI path
and find related repo configuration in `Storage`. Repo configuration
knows repo type (e.g. `maven` or `docker`) and storage settings for repo
(different repos may have different storage back ends).
After reading repo config it constructs new `Slice` for config
and proxies current request to this slice.

### Artipie architecture

Main components of Artipie software are:
 - Adapter: this component works with single binary artifact format, e.g.
 Maven-adapter or Docker-adapter. Adapter usually consist of two logical parts:
 front-end and back-end. The back-end of adapter works with binary artifacts
 and its metadata. It can be used independently as a library to store artifacts
 and generate metadata. It uses `Storage` from `artipie/asto` as a storage.
 Front-end of adapter implements `Slice` interface from `artipie/http` module.
 It handles incoming HTTP requests, process it using back-end objects, and
 generate HTTP responses.
 - Storage: Artipie uses abstract key-value storage `artipie/asto` in all modules.
 Storage support atomic transactional operations and it's thread safe.
 Artipie has multiple storage implementations: in-memory storage,
 file-system storage, AWS S3 storage. Storage can be used to store binary artifacts
 or for configuration files.
 - Artipie: configured assebmly of adapters. Artipie can be configured to read
 repository configuration files from the storage. Artipie can find configuration
 file by repository as a key name. Artipie implements `Slice` interface and can
 handle HTTP requests. It reads repository name from request URI path,
 finds configuration for adapter, constructs appropriate storage for adapter,
 and redirects the request to adapter.
 - Web server: any `Slice` implementation (Artipie or single module) can be used
 as a back-end for web server. We require the server to be reactive and to support
 non-blocking network IO operations. One of possible implementations is
 [vertx-server](https://github.com/artipie/vertx-server/).

Here is cross-module dependency diagram:

![diagram](/_docs/artipie-classes.png)

### Configuration

Artipie should be configured before startup.
Main meta configuration `yaml` file should contains storage config,
where adapter configuration files are located. Storage back-end
can be either `fs` or `s3`. File-system `fs` storage uses local
file system to store key-value data. AWS `s3` storage uses S3 cloud
service to store data in blobs.
```yaml
meta:
  # configuration storage
  storage:
    # storage type (either `fs` or `s3`)
    type: fs
    path: /artipie/storage
```

Meta storage contains adapters configuration, where key is a repository name,
and value is adapter config `yaml` file:
```text
config storage
├── maven-repo
├── docker-one
├── hello-npm
└── rpm
```
Each configuration file should specify what is the type of repository should be used
(adapter), and storage configuration (each repository may reference to different storage).
```yaml
repo:
  type:
    maven
  storage:
    type: s3
    url: s3://acme.com/snapshot
    username: admin
    password: 123qwe
```

### Deployment

Artipie web server can be started as standalone Java application, or
started in a cluster with multiple instance behind load balancer.
If Artipie was started in a cluster, all instances should receive
single meta configuration for Artipie module. It's recommended
that S3 storage be used for multi-instance deployment.

### How to start

To start Vertx server with Artipie service, you need to build package first:
`mvn clean package`; And start it with Java command then:
```bash
java -Dartipie.storage=/tmp/artipie -Dartipie.port=8080 \
  -jar ./target/artipie-jar-with-dependencies.jar
```
where `artipie.storage` system property is a location of root of configuration file-system storage,
`artipie.port` is a property for server HTTP port.

## How to contribute

Fork repository, make changes, send us a pull request. We will review
your changes and apply them to the `master` branch shortly, provided
they don't violate our quality standards. To avoid frustration, before
sending us your pull request please run full Maven build:

```
$ mvn clean install -Pqulice
```

To avoid build errors use Maven 3.2+.

Thanks to [FreePik](https://www.freepik.com/free-photos-vectors/party) for the logo.
