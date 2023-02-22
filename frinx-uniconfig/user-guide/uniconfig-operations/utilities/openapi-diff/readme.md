# Difference between OpenAPI specifications

## Introduction

User can find program for checking the difference between OpenAPI specifications in the utils/ directory
after building and unpacking the UniConfig distribution. User can use it with a shell script called
'show_swagger_diff.sh'.

This program uses [OpenAPI-diff](https://github.com/OpenAPITools/openapi-diff) for generating OpenAPI differences.

## Usage

Script ```./show_swagger_diff.sh``` contains four arguments. Each one has its own identifier so user can use any order of arguments.
Every argument is optional as the script has default values for each argument.

* ```--former, -f /path/to/former/yaml/files``` - optional argument. Path with former OpenAPI specifications (.yaml files). Default path is 'openapi_diff/old'.
* ```--new, -n /path/to/new/yaml/files``` - optional argument. Path with new OpenAPI specifications (.yaml files). Default path is 'openapi_diff/new'.
* ```--output, -o /path/to/output``` - optional argument. Path for the html output file with differences. Default path is 'openapi_diff'.
* ```-s``` - optional argument. Silent printing. Shows less information.

!!!
Bash script ```./show_swagger_diff.sh``` also includes simple help facility. There are two options how to show the help text:
1. ```./show_swagger_diff.sh -h```
2. ```./show_swagger_diff.sh --help```
!!!

!!!
Script only accepts YAML files.
!!!

## Example use-case

### Default Usage

This is basic usage of the script, with and without optional arguments. 
Open a terminal, go to the ../utils directory and run command:

```console
./show_swagger_diff.sh
```

OR

```console
./show_swagger_diff.sh -f openapi_diff/old -n openapi_diff/new -o openapi_diff
```

```console Output:
Directory openapi_diff/old and openapi_diff/new exist.
Starting openapi-diff for 'OpenAPI_specification_1'...
'OpenAPI_specification_1' was deleted in the new version.

Starting openapi-diff for 'OpenAPI_specification_2'...
Diff for 'OpenAPI_specification_2' successful

Starting openapi-diff for 'OpenAPI_specification_3'...
No differences. Specifications are equivalents. Skipping 'OpenAPI_specification_3'...

Open openapi_diff/openapi_diff.html in web browser to view all the differences between old and new modules.
```

### Usage with non-existent input path

This is basic usage of the script where some specified input directories do not exist.
Open a terminal, go to the ../utils directory and run command:

```console
./show_swagger_diff.sh -n non/existent/path
```

```console Output:
Error: Directory openapi_diff/old or non/existent/path does not exist.
```