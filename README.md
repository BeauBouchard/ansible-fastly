# ansible-fastly

[![Build Status](https://travis-ci.org/Jimdo/ansible-fastly.svg?branch=master)](https://travis-ci.org/Jimdo/ansible-fastly) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-Jimdo.fastly-blue.svg?style=flat)](https://galaxy.ansible.com/Jimdo/fastly/)

Ansible module to configure services in Fastly

## Installation

``` bash
$ ansible-galaxy install Jimdo.fastly
```

## Documentation

### Module options

| Name                 | Required | Description                                                                                   | Default |
|:---------------------|:---------|:----------------------------------------------------------------------------------------------|:--------|
| name                 | true     | The unique name for the service to create                                                     |         |
| fastly_api_key       | false    | Fastly API key. If not set then the value of the FASTLY_API_KEY environment variable is used. |         |
| activate_new_version | false    | Configures whether newly created versions should be activated automatically                   | true    |
| domains              | true     | List of domain names to serve as entry points for your service                                |         |
| backends             | true     | List of backends to service requests from your domains                                        |         |
| cache_settings       | false    | List of cache settings                                                                        |         |
| conditions           | false    | List of conditions                                                                            |         |
| gzips                | false    | List of gzip configurations                                                                   |         |
| headers              | false    | List of headers to manipulate for each request                                                |         |
| response_objects     | false    | List of response objects                                                                      |         |
| settings             | false    | Settings object                                                                               |         |

### Backend

[Fastly documentation](https://docs.fastly.com/api/config#backend)

| Field         | Required | Type                                                    | Default |
|:--------------|:---------|:--------------------------------------------------------|:--------|
| name          | true     | string                                                  |         |
| port          | false    | integer                                                 | 80      |
| address       | true     | string                                                  |         |
| ssl_hostname  | false    | string                                                  |         |
| ssl_ca_cert   | false    | string                                                  |         |

### Cache Settings

[Fastly documentation](https://docs.fastly.com/api/config#cache_settings)

| Field           | Required | Type                                                    | Default |
|:----------------|:---------|:--------------------------------------------------------|:--------|
| name            | true     | string                                                  |         |
| action          | false    | enum ('cache', 'pass', 'restart')                       |         |
| cache_condition | false    | string                                                  |         |
| stale_ttl       | false    | integer                                                 | 0       |

### Condition

[Fastly documentation](https://docs.fastly.com/api/config#condition)

| Field     | Required | Type                                                    | Default |
|:----------|:---------|:--------------------------------------------------------|:--------|
| name      | true     | string                                                  |         |
| comment   | false    | string                                                  |         |
| priority  | false    | integer                                                 | 0       |
| statement | true     | string                                                  |         |
| type      | true     | enum ('REQUEST', 'PREFETCH', 'CACHE', 'RESPONSE')       |         |

### Header

[Fastly documentation](https://docs.fastly.com/api/config#header)

| Field              | Required | Type                                                      | Default |
|:-------------------|:---------|:----------------------------------------------------------|:--------|
| name               | true     | string                                                    |         |
| action             | false    | enum ('set', 'append', 'delete', 'regex', 'regex_repeat') | set     |
| dst                | true     | string                                                    |         |
| ignore_if_set      | false    | integer (one of [0,1])                                    | 0       |
| priority           | false    | integer                                                   | 100     |
| regex              | false    | string                                                    |         |
| request_condition  | false    | string                                                    |         |
| response_condition | false    | string                                                    |         | 
| src                | true     | string                                                    |         |
| substitution       | false    | string                                                    |         |
| type               | true     | enum ('request', 'fetch', 'cache', 'response')            |         |

### Response Object

[Fastly documentation](https://docs.fastly.com/api/config#response_object)

| Field             | Required | Type                                                      | Default |
|:------------------|:---------|:----------------------------------------------------------|:--------|
| name              | true     | string                                                    |         |
| request_condition | false    | string                                                    |         |
| response          | false    | string                                                    | Ok      |
| status            | false    | integer                                                   | 200     |

### Settings

[Fastly documentation](https://docs.fastly.com/api/config#settings)

| Field               | Required | Type                                                      | Default |
|:--------------------|:---------|:----------------------------------------------------------|:--------|
| general.default_ttl | false    | integer                                                   | 3600    |


## Examples

### Using the fastly_service module in a Playbook

``` yml
---
- hosts: localhost
  connection: local
  gather_facts: False
  roles:
    - Jimdo.fastly
  tasks:
    - fastly_service:
        name: Redirect service
        domains:
          - name: test1.example.net
            comment: redirect domain
        backends:
          - name: localhost
            port: 80
            address: 127.0.0.1
        headers:
          - name: Set Location header
            dst: http.Location
            type: response
            action: set
            src: http://test3.example.net req.url.path
            ignore_if_set: 0
            priority: 10
        response_objects:
          - name: Set 301 status code
            status: 301
```

``` bash
$ ansible-playbook -i localhost, fastly.yml
```
