#BOB the buildah

[![Build Status](https://travis-ci.org/balanced/bob.svg)](https://travis-ci.org/balanced/bob)
![Alt text](http://i49.photobucket.com/albums/f265/jsh00ter12/jfkl.jpg)

Builds debian packages via fpm and pip.

## Usage

To build an ubuntu debian from git@github.com://balanced/balanced.git for tag
1.8.1 run

`bob build ubuntu balanced balanced 1.8.1`

## TODO:

* wrap in an http api and accept hooks from github and travis

## Configure a project

Bob uses a YAML file (`build.yml`) inside the project it's building. Here's
how it should look:

```yaml
---
# which version of the settings to use.
version: 1
description: my sexy project
targets:
  # ubuntu is currently the only target
  ubuntu:
    # do not package anything that matches these globs
    exclude:
      - "*.pyc"
      - ".git*"
    # required to build this project
    build_dependencies:
      - libpq-dev
      - python-dev
      - libxml2-dev
      - libxslt1-dev
    # required to install this package
    dependencies:
      - libxml2
      - libxslt1.1
      - libpq5
      - ipython
    # any executable scripts relative to the root of this project that will be
    # executed upon package action
    before_install:
    after_install:
      - scripts/after-install.sh
    before_remove:
      - scripts/before-remove.sh
    after_remove:
    # what to do with the finished product (s3 and depot are the only actions
    # right now)
    destinations:
#      s3:
#        destination: s3://apt.vandelay.io
      depot:
        destination: s3://apt.vandelay.io
        gpg_key: 277E7787
        component: unstable
        codename: lucid
    # when and where to tell the world about builds
    notifications:
      hipchat:
        room_id: dev
        color: purple
        on:
        - success
        - failure
```

You'll also need to configure travis to call out to bob when it finishes. In
`.travis.yml`:

```yaml
notifications:
  webhooks:
    urls:
      - https://builder.vandelay.io/hooks/travis
    on_success: always
    on_failure: never
    on_start: false
```

## Hacking on it

* Clone https://github.com/balanced/bob
* Clone https://github.com/balanced-cookbooks/ba-bob
* set the env variables specified in ba-bob/.kitchen.yml
* `bundle exec kitchen converge`
* `bundle exec kitchen login`
* `time bob build ubuntu balanced balanced 153f85e`
* hang out for 10 minutes.

## Run via uwsgi

`bobd -u uwsgi.ini -- --env=BOB_CONF=xxx.ini`

## Manually kicking off a build

```bash
curl https://builder.vandelay.io/hooks/travis --data-binary  '{"branch": "1.4.14", "commit": "1.4.14", "repository": {"name": "digi", "owner_name": "balanced"}, "result_message": "Passed"}
```
