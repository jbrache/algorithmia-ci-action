# algorithmia-ci action
An algorithmia github Action to test and deploy github backed Algorithmia.com algorithms. 
When attached as a workflow for an Algorithmia algorithm, can automatically deploy new versions when provided tests pass.

For this action to function, the github action `actions/checkout` must be called before this action, as we search for key files in your algorithm repository for simplicity.
An example workflow of this is below, however if you want to jump into a full algorithm example click [here](https://github.com/algorithmiaio/algorithmia_ci).


# Action Input

```
inputs:
  api_key:
    description: 'Your algorithm management capable Algorithmia API key'
    required: true
  api_address:
    description: 'The API address for the Algorithmia Cluster you wish to connect to'
    required: false
    default: 'https://api.algorithmia.com'
  version_schema:
    description: 'identifier to describe how to promote this release'
    required: false
    default: "minor"
  path:
    description: 'directory name for the local project stored in /github/workspace due to an "actions/checkout" command'
    required: true
  test_algo:
    description: 'true or false string whether to test an algo with a sample input'
    required: false
    default: true
```

```
  api_key - (required) - An Algorithmia api key that has execute access for the algorithm you wish to test, has full access for algorithms. read more about that [here](https://algorithmia.com/developers/platform/customizing-api-keys)
  api_address - (optional) - The Algorithmia API cluster address you wish to connect to, if using a private cluster; please provide the correct path to your environment.
  version_schema - (optional) - The [semantic version](https://semver.org/) parameter that will get incremented whenever this action gets triggered. May be "Major", "Minor", or "Revision"
  path - (required) - The workspace path variable provided to the actions/checkout@v2 of the current job

```

# Important Files
Your Algorithm naturally contains a `algorithmia.conf` file which we parse to gather the algorithm name from, however a new file called `TEST_CASES.json` is required.
The required schema of this json file is shown below:

# Case Schema
Your test cases should follow the following json schema
```
[
 { 
    "case_name": String,
    "input": Any,
    "expected_output": Any
  },
  ...
]
```

`input` and `expected_output` will be expected to be the raw input/output that the algorithm you're testing should expect.


# Example workflow
Below is the standard algorithm CI workflow we recommend you use to take advantage of our CI/CD system.
If you have any questions or comments please feel free to create an issue.

```
# This is an example using the Algorithmia CI action

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request
# In this instance we're triggering this workflow whenever a push to master is performed.
on:
  push:
    branches: [master]

jobs:
  # This workflow only contains a single job, however if you'd like you can split this processing across multiple jobs.
  algorithmia-ci:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2.0.0
      with:
        ref: ${{github.sha}}
        path: algorithm
    - name: Algorithmia CI
      uses: algorithmiaio/algorithmia-ci-action@v1.0.1
      with:
        # Your master Algorithmia API key
        api_key: ${{ secrets.mgmt_api_key }}
        # The API address for the Algorithmia Cluster you wish to connect to
        api_address: https://api.algorithmia.com
        # identifier to describe how to promote this release ('major', 'minor', 'revision')
        version_schema: minor
        # the path variable you defined in the actions/checkout action triggered before this one.
        path: algorithm
```
