# Storages

## Overview

This library provides a generic interface for interfacing multiple file-storage:

* local
* storage on remote server over ssh
* AWS S3 (https://aws.amazon.com/s3/)
* OpenStack Object Storage service Swift (https://docs.openstack.org/api-ref/object-store/)

## Installation

The full code is available in this repository, the `systran` direcctory contains the library. `cli/storages-cli.py` is a commandline utility allowing to test easily the connections to the different services.

Alternatively, you can install both from `PyPi` with `pip install systran.storages`.

## Features

For each service, the library provides the following functions:

* `get_file(remote_path, local_path)`: download a remote file to a local file. If the file is already present (based on checksum and/or date and/or file size), the local file is not modified.
* `get_directory(remote_path, local_path)`: recursively download all the files from a given directory.
* `stream(remote_path, buffer_size=1024)`: stream the content of the file through a generator function in `buffer_size` packets. If the service does not support streaming, file is first downloaded locally to a temporary directory then streamed.
* `push(local_path, remote_path)`: push a local file to a remote location
* `listdir(remote_path, recursive=False)`: list of the files and directory at the `remote_path` location
* `delete(remote_path, recursive=False)`: delete a file or a directory
* `rename(remote_path, new_remote_path)`: rename a remote_file
* `exists(remote_path)`: check if a remote file or directory exists

## Configuration

Configuration of the services is done with a json dictionary: `{"key1": DEF1, "key2": DEF2, ..., "keyN": DEFN}`, where `key1`...`keyN` are the identifiers of each storage, and `DEF1`...`DEFN` their configuration.

Configuration `DEFI` are themselves json dictionary with following fields:
```json
{
	"description": "description of the storage (optional)",
	"type": "local|ssh|s3|swift|http",
	/STORAGE OPTIONS/
}
```

The dictionary is passed to the `StorageClient` constructor:

```python
from systran.storage import StorageClient

client = StorageClient(services)
```

## Utilisation

The different services are method of the `client` object where `remote_path` is a string with 2 fields: `storage:path`. `storage` is the identifierr of the storage as defined in the configuration dictionary. `path` is the local path.

Paths use `/` delimiter. All path starts at `/` and relative path accessor `.`, `..` are not supported. For the storages with actual directory structure (S3, Swift, ...), a hierarchical file organization is simulated.

## Services

### Local

Type of the service is `local`. 

_Storage options_:

* (optional, default `'/'`) `basedir`: defines the base directory where files are stored.

### SSH

Type of the service is `ssh`.

_Storage options_:

* (required) `server`: name of IP of the remote server
* (option, default 22) `port`: name of the port to connect to
* (required) `user`: login to connect with on the server
* (optional) `password`: user password, preferably use `pkey`
* (optional) `pkey`: private key used to connect on the service. The value does not include head (`-----BEGIN RSA PRIVATE KEY-----`) and tail (`-----END RSA PRIVATE KEY-----`)
* (optional, default user home) `basedir`: defines the base directory where files are stored.

### S3

Type of the service is `s3`.

_Storage options_:

* bucket_name, access_key_id=None, secret_access_key=None,
                 region_name=None, assume_role=None, transfer_config=None

### Swift

Type of the service is `swift`.

_Storage options_:

* container_name, auth_config=None, transfer_config=None

### HTTP

Type of the service is `http`.

_Storage options_:

* pattern_get, pattern_push=None, pattern_list=None
