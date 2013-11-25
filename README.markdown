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

`git rm` the 1.* file you don't need. Also `git rm` the README.md file.

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
git pull -X ours discourse master

~~git fetch discourse~~

```

Either decide on a release, or go with the latest.

```bash
~~git checkout -b mergetemp v0.9.7.3
or
git checkout -b mergetemp latest-release
git checkout master~~

```

## Summary

My current build steps are:

```bash
cd discourse
git remote add upstream -m master git@github.com:NeilBryant/discourse-quickstart-neb.git
git pull -s recursive -X theirs upstream master
git rm 1.install-rbenv
git commit -a -m "Removed rbenv"
git commit -a -m "Added production.rb"
git push
```
Go log on (probably don't need this if you yank the discourse rakefile call)

```bash
git remote add discourse git@github.com:discourse/discourse.git
git pull -X ours discourse master
git pull -s recursive -X theirs upstream master
git push

```

Log on. Manually:

```bash
$rinit
ruby --version
\# ruby 1.9.3p125 (2012-02-16 revision 34643) [x86_64-linux]
gem --version
\# 1.8.28
cd app-root/repo
gem install bundler
gem install libv8
gem install therubyracer
bundle install  --no-deployment
```

This is because libv8/therubyracer doesn't seem to install in 
As an alternate, you might try changing the Gemfile to rubyracer 0.10.x before git pushing it.

When the system runs out of space, run this:

```bash
#!/bin/bash

# For bundle install --no-deployment
cachedir=~/app-root/data//.rvm/gems/ruby-1.9.3-p125/cache
sourcedir=~/app-root/data//.rvm/gems/ruby-1.9.3-p125/gems

# For bundle install --deployment
# cachedir=~/app-root/repo/vendor/bundle/ruby/1.9.1/cache
# sourcedir=~/app-root/repo/vendor/bundle/ruby/1.9.1/gems

for f in $cachedir/*.gem
do
  echo "Processing $f file..."
  # take action on each file. $f store current file name
  # This would keep the path intact...
  # rm -rf ${f/.gem/}
  rm -rf $sourcedir/`basename "$f" .gem`
done

rm -rf ~/app-deployments/2*
rm -rf ~/app-deployments/2013-11-25_01-25-59.907/
```

  252  RUBY_GC_MALLOC_LIMIT=90000000 rake db:migrate

  253  RUBY_GC_MALLOC_LIMIT=90000000 rake db:migrate --trace


then continue
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