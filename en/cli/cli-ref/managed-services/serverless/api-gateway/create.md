---
editable: false
sourcePath: en/_cli-ref/cli-ref/managed-services/serverless/api-gateway/create.md
---

# yc serverless api-gateway create

Create API Gateway

#### Command Usage

Syntax: 

`yc serverless api-gateway create <API-GATEWAY-NAME> [Flags...] [Global Flags...]`

#### Flags

| Flag | Description |
|----|----|
|`--name`|<b>`string`</b><br/>Api-gateway name.|
|`--async`|Display information about the operation in progress, without waiting for the operation to complete.|
|`--description`|<b>`string`</b><br/>Api-gateway description.|
|`--labels`|<b>`key=value[,key=value...]`</b><br/>A list of label KEY=VALUE pairs to add. For example, to add two labels named 'foo' and 'bar', both with the value 'baz', use '--labels foo=baz,bar=baz'.|
|`--spec`|<b>`string`</b><br/>Api-gateway specification file name.|
|`--network-name`|<b>`string`</b><br/>Api-gateway network name.|
|`--network-id`|<b>`string`</b><br/>Api-gateway network id.|
|`--subnet-name`|<b>`value[,value]`</b><br/>Api-gateway subnet names.|
|`--subnet-id`|<b>`value[,value]`</b><br/>Api-gateway subnet ids.|
|`--no-logging`|Disable logging from api-gateway.|
|`--log-group-id`|<b>`string`</b><br/>Send logs to custom log group by id.|
|`--log-group-name`|<b>`string`</b><br/>Send logs to custom log group by name.|
|`--log-folder-id`|<b>`string`</b><br/>Send logs to default log group of custom folder by id.|
|`--log-folder-name`|<b>`string`</b><br/>Send logs to default log group of custom folder by name.|
|`--min-log-level`|<b>`string`</b><br/>Min log level. Values: 'trace', 'debug', 'info', 'warn', 'error', 'fatal'|

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
