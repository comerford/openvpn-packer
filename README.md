# openvpn-packer 🚪📦 #

[![GitHub Build Status](https://github.com/cisagov/openvpn-packer/workflows/build/badge.svg)](https://github.com/cisagov/openvpn-packer/actions)

This project uses [packer](https://packer.io)
to create an AMI with [OpenVPN](https://openvpn.net)
installed and configured for use as a VPN gateway.
It uses the
[ansible-role-openvpn](https://github.com/cisagov/ansible-role-openvpn)
role to install OpenVPN.

## Pre-requisites ##

This project requires a build user to exist in AWS.  The accompanying terraform
code will create the user with the appropriate name and permissions.  This only
needs to be run once per project, per AWS account.  This user will also be used by
GitHub Actions.

```console
cd terraform
terraform init --upgrade=true
terraform apply
```

Once the user is created you will need to update the
[repository's secrets](https://github.com/cisagov/openvpn-packer/settings/secrets)
with the new encrypted environment variables.

```console
terraform state show module.iam_user.aws_iam_access_key.key
```

Take the `id` and `secret` fields from the above command's output and create the
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables in the
[repository's secrets](https://github.com/cisagov/openvpn-packer/settings/secrets).

## Building the Image ##

### Using GitHub Actions ###

1. Create a [new release](https://help.github.com/en/articles/creating-releases)
   in GitHub.
1. There is no step 2!

GitHub Actions can build this project in three different modes depending on
how the build was triggered from GitHub.

1. **Non-release test**: After a normal commit or pull request GitHub Actions
   will build the project, and run tests and validation on the
   packer configuration.  It will __not__ build an image.
1. **Pre-release deploy**: Publish a GitHub release
   with the "This is a pre-release" checkbox checked.  An image will be built
   and deployed using the [`prerelease`](.github/workflows/prerelease.yml)
   workflow.  This should be configured to deploy the image to a single region
   using a non-production account.
1. **Production release deploy**: Publish a GitHub release with
   the "This is a pre-release" checkbox unchecked.  An image will be built
   and deployed using the [`release`](.github/workflows/release.yml)
   workflow.  This should be configured to deploy the image to multiple regions
   using a production account.

### Using Your Local Environment ###

Packer will use your
[standard AWS environment](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)
to build the image.

The [Packer template](src/packer.json) requires one environment variable to be defined:

- `BUILD_REGION`: the region in which to build the image.

Additionally, the following optional environment variables can be used
by the [Packer template](src/packer.json) to tag the final image:

- `GITHUB_IS_PRERELEASE`: boolean pre-release status
- `GITHUB_RELEASE_TAG`: image version
- `GITHUB_RELEASE_URL`: URL pointing to the related GitHub release

Here is an example of how to kick off a pre-release build:

```console
pip install --requirement requirements-dev.txt
ansible-galaxy install --force --force-with-deps --role-file src/requirements.yml
export BUILD_REGION="us-east-2"
export GITHUB_RELEASE_TAG=$(./bump_version.sh show)
echo "us-east-2:alias/cool/ebs" | ./patch_packer_config.py src/packer.json
packer build --timestamp-ui src/packer.json
```

If you are satisfied with your pre-release image, you can easily create a release
that deploys to all regions by changing the input to
`patch_packer_config.py` to include additional comma-separated regions:kms_keys
and rerunning packer:

```console
echo "us-east-1:alias/cool/ebs,us-east-2:alias/cool/ebs,us-west-1:alias/cool/ebs,\
us-west-2:alias/cool/ebs" | ./patch_packer_config.py src/packer.json
packer build --timestamp-ui src/packer.json
```

See the patcher script's help for more information about its options and
inner workings:

```console
./patch_packer_config.py --help
```

## Contributing ##

We welcome contributions!  Please see [here](CONTRIBUTING.md) for
details.

## License ##

This project is in the worldwide [public domain](LICENSE).

This project is in the public domain within the United States, and
copyright and related rights in the work worldwide are waived through
the [CC0 1.0 Universal public domain
dedication](https://creativecommons.org/publicdomain/zero/1.0/).

All contributions to this project will be released under the CC0
dedication. By submitting a pull request, you are agreeing to comply
with this waiver of copyright interest.
