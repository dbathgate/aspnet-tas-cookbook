# CF CLI

* The CF CLI is a command line interface for CloudFoundry

## [Installation](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

## Logging in

### Username/Password login

```bash
cf login -a https://api.sys.my.org
# Enter username and password when prompted
```

### SSO Login
```bash
cf login -a https://api.sys.my.org --sso
# Go to URL when prompted and copy passcode, paste into this terminal
```

### Skip SSL Validation
```bash
cf login -a https://api.sys.my.org --skip-ssl-validation
```

## Useful commands

### Push
* How to deploy applications
```bash
# if the manifest.yml exists in the working directory
cf push

# if the manifest.yml exists in a different directory
cf push -f some/other/dir/manifest.yml

# without a manifest
cf push my-app --stack windows --buildpack hwc_buildpack
```

### Logs
```bash
# actively tails and displays current logs
cf logs my-app

# gets the most recent logs
cf logs my-app --recent
```

### List resources
```bash
# list apps
cf apps

# list services
cf services

# list routes
cf routes
```

### Switch orgs/spaces

```bash
# Switches to the dev space
cf target -s dev

# Switches to a different org's stage space
cf target -o app2 -s stage
```