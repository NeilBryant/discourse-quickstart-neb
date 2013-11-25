# Discourse on Openshift

Installer and action hooks for Discourse on Openshift. Includes scripts to install Ruby 2.0.0-p0 with rbenv.

## Step by Step

### 01. Build the Discourse Gear

#### Create

These are the contents of my 'rhccreate.bash' file:

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

Alternatively, if you've already built the Gear, you can just clone it:

```bash
rhc git-clone -a discourse
```

#### Configure

```
cd discourse

git remote add upstream -m master git@github.com:NeilBryant/discourse-quickstart-neb.git
git pull -s recursive -X theirs upstream master

```

Before deployment update config/environments/production.rb.openshift with valid SMTP credentials.

In mycase, I use this command from the host PC:

```
scp -F tmp/vagrant-ssh-config tmp/production.rb.openshift  default:~/discourse/config/environments/production.rb.openshift

```

`git rm` the 1.* file you don't need.

Now deploy it to the Openshift Gear:

```
git push
```

### 02. Login to the Gear

```bash
rhc ssh -a  discourse
```

Openshift doesn't have a lot in the way of modifying your environment. The script configures an environmnt variable to use as a shortcut; so to start rbenv in your new ssh session, type:

```bash
$rinit
```

Make sure the version is correct, and install bundler:

```bash
ruby --version
gem install bundler
```

### 03. More Git Work (local)
```bash
git commit -a -m "Added Openshift hooks."
git remote add discourse git@github.com:discourse/discourse.git
git pull -X theirs discourse master

~~git fetch discourse~~

```

Either decide on a release, or go with the latest.

```bash
~~git checkout -b mergetemp v0.9.7.3
or
git checkout -b mergetemp latest-release
git checkout master~~

```

- \# try tag 'latest-release'
- git checkout v0.9.6.4
- \# git checkout -b tempbranch v0.9.6.4
- \# list tags with 'git tag -l'
- \# git checkout -b mergetemp tags/v0.9.7.3
- \# git checkout -b mergetemp tags/latest-release
- git checkout v0.9.6.4
- \# git checkout -b tempbranch v0.9.6.4
- \# list tags with 'git tag -l'
- \# git checkout -b mergetemp tags/v0.9.7.3
- \# git checkout -b mergetemp tags/latest-release

- \# ContinueOrExit "checkout master"
- \# git checkout master
- \# ContinueOrExit "checkout merge"
- \# git merge tempbranch

- \# ContinueOrExit "delete mergetemp"
- \# \# Remove mergetemp
- \# git branch -d tempbranch