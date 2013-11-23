# Discourse on Openshift

Installer and action hooks for Discourse on Openshift. Includes scripts to install Ruby 2.0.0-p0 with rbenv.

## Step by Step

### 01. Build the Discourse Gear

```bash
un=whoever
openshiftpwd=whatever
appname=discourse

set -x

export GIT_MERGE_AUTOEDIT=yes

rhc setup  -l $un -p $openshiftpwd --create-token
ssh-keyscan -H rhcloud.com >> ~/.ssh/known_hosts

# Create App
rhc app create $appname diy-0.1 -l "$un" -p "$openshiftpwd" || exit 1 | tee ~/openshiftappbuild.txt

rhc cartridge add postgresql-9.2 -a $appname | tee -a ~/openshiftappbuild.txt
rhc add-cartridge http://cartreflect-claytondev.rhcloud.com/reflect?github=smarterclayton/openshift-redis-cart -a $appname | tee -a ~/openshiftappbuild.txt

```

### 02. Login to the Gear

```bash
% RHC ssh -a  discourse
```

### 03. Gear --> Install rbenv
```bash
% $rinit
```

