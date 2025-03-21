---
editable: false
---

# yc dataproc job create-hive

Create a Dataproc Hive job.

#### Command Usage

Syntax: 

`yc dataproc job create-hive [Flags...] [Global Flags...]`

Aliases: 

- `submit-hive`

#### Flags

| Flag | Description |
|----|----|
|`--cluster-id`|<b>`string`</b><br/>ID of the cluster.|
|`--cluster-name`|<b>`string`</b><br/>Name of the cluster.|
|`--name`|<b>`string`</b><br/>Optional job name|
|`--query-file-uri`|<b>`string`</b><br/>Query file URI|
|`--query-list`|<b>`value[,value]`</b><br/>Query List|
|`--continue-on-failure`|Continue on query failure|
|`--script-variables`|<b>`stringToString`</b><br/>Script variables|
|`--properties`|<b>`stringToString`</b><br/>Properties|
|`--jar-file-uris`|<b>`value[,value]`</b><br/>JAR file URIs|
|`--async`|Display information about the operation in progress, without waiting for the operation to complete.|

#### Global Flags

| Flag | Description |
|----|----|
|`--profile`|<b>`string`</b><br/>Set the custom configuration file.|
|`--debug`|Debug logging.|
|`--debug-grpc`|Debug gRPC logging. Very verbose, used for debugging connection problems.|
|`--no-user-output`|Disable printing user intended output to stderr.|
|`--retry`|<b>`int`</b><br/>Enable gRPC retries. By default, retries are enabled with maximum 5 attempts.<br/>Pass 0 to disable retries. Pass any negative value for infinite retries.<br/>Even infinite retries are capped with 2 minutes timeout.|
|`--cloud-id`|<b>`string`</b><br/>Set the ID of the cloud to use.|
|`--folder-id`|<b>`string`</b><br/>Set the ID of the folder to use.|
|`--folder-name`|<b>`string`</b><br/>Set the name of the folder to use (will be resolved to id).|
|`--endpoint`|<b>`string`</b><br/>Set the Cloud API endpoint (host:port).|
|`--token`|<b>`string`</b><br/>Set the OAuth token to use.|
|`--format`|<b>`string`</b><br/>Set the output format: text (default), yaml, json, json-rest.|
|`-h`,`--help`|Display help for the command.|
