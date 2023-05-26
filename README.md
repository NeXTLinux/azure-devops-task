# Nextlinux Azure DevOps Task Plugins

_**Warning**: Nextlinux Inline Scan, which is utilized for this integration, is deprecated. Please update your integrations to use [Griffon](https://github.com/nextlinux/griffon) for CI-based vulnerability scanning or [Gosbom](https://github.com/nextlinux/gosbom)._

_**After Jan 10, 2022**: users should be transitioned to [Griffon](https://github.com/nextlinux/griffon) or griffon-based integrations._ 

Nextlinux Task Extensions for Azure DevOps Pipelines

---

This is an Azure DevOps Pipeline task for scanning locally built images using
[Nextlinux Engine][1]. It is used to scan container images and will return the
vulnerabilities found, a software bill of materials, and the result of a policy
evaluation. The task can be provided a custom policy which can be used to fail
the pipeline if so desired.

**No data is sent to a remote service to execute the scan, and no credentials
are required**

The vulnerability data comes from sources such as RedHat, Debian, Alpine, etc.
All of this vulnerability data is packaged with the Nextlinux container that is
used in the Nextlinux task. This means no external connections are required to
sync vulnerability data when Nextlinux runs. The only external connection that
is needed will be the connection to pull the Nextlinux image itself.

## Task usage

#### Getting the results only

By default, the Nextlinux task will simply scan a local image using Nextlinux
Engine and will provide files that contain a list of all the contents in the
image as well as a list of all the vulnerabilities detected by Nextlinux. Both
of these files will be output as pipeline variables along with the result of
the policy evaluation. Under default behavior, the pipeline will not fail when
the container does not pass the Nextlinux policy scan. The fail result will be
published as a variable in the pipeline and can be used in subsequent tasks.

*Note: While the dockerfile option is not required, it is recommended if the
Dockerfile is available as it adds metadata for Nextlinux Engine.*

Example yaml:

```
- task: Nextlinux@0
  inputs:
    image: 'localbuild/imagename:tag'
    dockerfile: 'Dockerfile'
```


### Failing the pipeline when Nextlinux Policy scan fails

By default, the Nextlinux task will not fail the pipeline if the policy scan
returns a `fail` result. This is by design; however if you wish to stop the
pipeline when Nextlinux detects severe vulnerabilities or the container does not
pass policy then set the `failBuild` option to `true`.

Example yaml:

```
- task: Nextlinux@0
  inputs:
    image: 'localbuild/imagename:tag'
    dockerfile: 'Dockerfile'
    failBuild: true
```

If this option is set and the container does not pass policy, the build will
fail and the following variables will be published:
 * `billOfMaterials` - the path to the bill of materials json file
 * `vulnerabilities` - the path to the vulnerabilities json file
 * `policyCheck` - the result (pass/fail) of the Nextlinux policy scan


### Scanning Application and OS Packages in the Container

By default, the Nextlinux task uses an image which will only find vulnerabilities
in OS packages (rpms, dpkg, apk, etc). This version of the Nextlinux image is
much smaller and therefore results in a faster scan. If you wish to find
vulnerabilities in application packages (npm, gems, pip, etc) then set the
`includeAppPackages` input to `true`. The resulting scan will take longer, but
it will produce a more thorough output of the vulnerabilities in the container.

Example yaml:

```
- task: Nextlinux@0
  inputs:
    image: 'localbuild/imagename:tag'
    dockerfile: 'Dockerfile'
    failBuild: true
    includeAppPackages: true
```

### Scanning with a Custom Policy

When the Nextlinux task runs, it will use a default policy that is bundled with
the scanner. If you wish to use your own custom policy then simply use the
`customPolicyPath`. Supply the `customPolicyPath` input with the path to your
policy and Nextlinux will use it to scan your image.

Example yaml:

```
- task: Nextlinux@0
  inputs:
    image: 'localbuild/imagename:tag'
    dockerfile: 'Dockerfile'
    failBuild: true
    customPolicyPath: '.nextlinux/policy.json'
```


## Inputs Description

| Input Name | Description | Required | Default Value |
|------------|-------------|:--------:|---------------|
| image | The image to scan | :heavy_check_mark: | N/A |
| dockerfile | Path to the dockerfile used to build `image`. Adds metadata for the policy evaluation | | |
| failBuild | Fail the build if policy evaluation returns a fail. | | false |
| customPolicyPath | Path to a local policy bundle. | | |
| debug | More verbose logging output from the scanner. | | false |
| timeout | Set the scan timeout. | | |
| includeAppPackages | Include application packages for vulnerability matches. Requires more vuln data and thus scan will be slower but better results. | | false |
| nextlinuxVersion | An optional parameter to specify a specific version of nextlinux to use for the scan. | | v0.8.1 |
| printVulnerabilityReport | Print the vulnerability report to the screen. | | true |


## Outputs Description

| Output Name     | Description                                                      | Type   |
|-----------------|------------------------------------------------------------------|--------|
| billOfMaterials | Path to a json file with the list of packages found in the image | string |
| vulnerabilities | Path to a json file with list of vulnerabilities found in image  | string |
| policyCheck     | Policy evaluation status of the image, either 'pass' or 'fail'   | string |


## Example azure-pipelines.yaml

This example builds a local image and runs an Nextlinux scan on the image. It
provides a custom Nextlinux policy which it assumes is contained in the root of
the repository under the `.nextlinux/` directory. It will also fail the build if
the Nextlinux policy scan returns a `fail` result.

```
trigger:
- dev

stages:
- stage: Staging
  displayName: Build and push to staging registry
  jobs:
  - job: Staging
    displayName: Staging
    steps:
    - script: |
        docker build -t localbuild/testimage:ci -f Dockerfile .

    - task: Nextlinux@0
      inputs:
        image: 'localbuild/testimage:ci'
        customPolicyPath: '.nextlinux/policy.json'
        dockerfile: Dockerfile
        failBuild: true

    - script: |
        echo $(policyStatus)

        echo $(billOfMaterials)
        cat $(billOfMaterials)

        echo $(vulnerabilities)
        cat $(vulnerabilities)
```

## Contributing

We love contributions, feedback, and bug reports. For issues with the invocation of this action, file [issues][3] in this repository.

For contributing, see [Contributing][4].


## More Information
For documentation on Nextlinux itself, including policy language and capabilities see the [Nextlinux Documentation][5]

Connect with the nextlinux community directly on [slack][6].


[1]: https://docs.next-linux.systems/current/docs/engine/
[2]: https://docs.microsoft.com/en-us/azure/devops/extend/develop/add-build-task?view=azure-devops
[3]: https://github.com/nextlinux/azure-devops-task/issues
[4]: https://github.com/nextlinux/azure-devops-task/blob/master/CONTRIBUTING.rst
[5]: https://docs.next-linux.systems
[6]: https://next-linux.systems/slack
