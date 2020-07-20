# bbtrigger
A small python tool that uses the Bitbucket Cloud REST API to trigger pipelines.

## Requirements
- python3
- `requests` pip module
- bitbucket cloud app password/token with necessary permissions

## Info
You might find this useful if you need to programmatically trigger
a pipeline e.g. from within a script or a different repo/pipeline.

Supports pipelines associated with Bitbucket branches, as well as
custom pipelines. For custom pipelines you can also specify custom
variables, both secure (masked) and standard key:value (non-masked).

To set a variable as secure ':s' is appended: `--var key:value:s`

This script uses Bitbucket's app authorisation tokens for authentication.
You can input this interactively by only specifying your username
with `--user`. Alternatively, specify both username and password/token
if run programmatically: `--user username:token`.

Use `--dryrun` to test the API payload first if you wish.

The following environment variables can be used instead of providing args:
```
BB_USER
BB_WORKSPACE
BB_REPO
BB_BRANCH
```

## Usage
### Pipeline associated with a branch
For example, you have the following pipeline:
```
pipelines:
  branches:
    dev:
      - step:
          <<: *build
      - step:
          <<: *mkartifact
      - step:
          <<: *upload
          deployment: qa-upload
      - step:
          <<: *codedeploy
          deployment: qa-deploy
```
Dryrun mode:
```
[elpy@testbox bbtrigger]$ bbtrigger --user user:token --workspace myorg --repo myrepo --branch dev --dryrun

url: https://api.bitbucket.org/2.0/repositories/myorg/myrepo/pipelines/
user: user
payload:
{
    "target": {
        "ref_type": "branch",
        "type": "pipeline_ref_target",
        "ref_name": "dev"
    }
}
```
To trigger the pipeline:
```
[elpy@testbox ~]$ bbtrigger --user user:token --workspace myorg --repo myrepo --branch dev
bbtrigger: success: response 201 for url: https://api.bitbucket.org/2.0/repositories/myorg/myrepo/pipelines/
```

### Custom pipeline
Using the below custom pipeline as an example:
```
pipelines:
  custom:
    nightly:
      - variables:
          - name: key1
          - name: key2
          - name: key3
      - step:
          name: nightly build pipeline
          script:
            - packer validate packer1.json
            - scripts/build.sh packer1.json
            - scripts/post-build.sh
          artifacts: [build.log]
```


Dryrun mode:
```
[elpy@testbox ~]$ bbtrigger --user user:token --workspace myorg --repo myrepo --branch master --custom nightly --var key1:value1 --var key2:value2:s --var key3:value3:s --deployment --dryrun 

url: https://api.bitbucket.org/2.0/repositories/myorg/myrepo/pipelines/
user: user
payload:
{
    "target": {
        "ref_type": "branch",
        "type": "pipeline_ref_target",
        "ref_name": "master",
        "selector": {
            "type": "custom",
            "pattern": "nightly"
        }
    },
    "variables": [
        {
            "key": "key1",
            "value": "value1"
        },
        {
            "key": "key2",
            "value": "value2",
            "secured": true
        },
        {
            "key": "key3",
            "value": "value3",
            "secured": true
        }
    ]
}
```
To trigger the pipeline remove `--dryrun`:
```
[elpy@testbox ~]$ bbtrigger --user user:token --workspace myorg --repo myrepo --branch master --custom nightly --var key1:value1 --var key2:value2:s --var key3:value3:s
bbtrigger: success: response 201 for url: https://api.bitbucket.org/2.0/repositories/myorg/myrepo/pipelines/
```

