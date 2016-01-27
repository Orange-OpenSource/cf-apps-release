# cf-apps-release
A generic bosh release to deploy apps on CF 

# Mapping bosh concepts to CF concepts

# Sample manifest

```yml
jobs: 
- name: autosleep-app
  templates: 
  - name: cf-app
    release: cf-apps-release
  instances: 2
  resource_pool: tiny
  properties:
    env:
      - JAVA_OPTS: -verbosegc
    bound-services:
      - autosleep-mysql

- name: autosleep-mysql
  templates: 
  - name: cf-service
    release: cf-apps-release
  instances: 2
  resource_pool: tiny
  properties:
    type: p-mysql    
    plan: default    
    arbitrary params: "{ action=update}"    

properties:
```

# Sample usage and life cycle

```sh
#Initial deployment
$ bosh deploy
deploying autosleep-app/1 
[cf push autosleep-v105-i1 -i $cf_instance_per_ambassador -p /var/vcap/jobs/../autosleep.jar]  
deploying autosleep-mysql/1 
[cf create-service autosleep-mysql p-mysql default]  

# Troubleshoot failed deployment through logs
$ bosh logs autosleep-app/1 

# App version update
# Version upgrade with 20% slices: canary=1 instance=4 cf_instance_per_ambassador=1
# Concourse job has created a new release with updated job source/blob: autosleep.jar
$ bosh upload release
# Operator chooses to deploy new version
$ bosh deploy 
updating autosleep-app/1
updating autosleep-app/2
updating autosleep-app/3
updating autosleep-app/4

# service update: autosleep-mysql plan update
# edit manifest with autosleep-mysql.plan: 1GB
$ bosh deploy
- autosleep-mysql.plan:100 MB
+ autosleep-mysql.plan:1 GB
Are you sure you want to apply these changes ?
Y
Updating job autosleep-mysql/1
[cf update-service autosleep-mysql -p 1GB] 

# service update: add new a service: redis 
# Version rollback
# Configuration change

# Scale out app
# edit manifest with cf_instance_per_ambassador: 6
$ bosh deploy
- cf_instance_per_ambassador: 5
+ cf_instance_per_ambassador: 6
Are you sure you want to apply these changes ?
Y
Updating job autosleep-app/1
Updating job autosleep-app/2

# Snapshot
$ bosh take snapshot
# delete app
# 

```

# Pending issues

* synchronization among job instances:
   * create service
   * bind service
   * create route

Solutions:
* no synchronization and rely on CF CLI idempotences/ at least once, and ignore extra parasite error traces
* test for existence of actions before creating them
* actually synchronize among job instances
   * serialize CC API calls ?
   * add messaging among cf-app/cf-service 
* rely on monit restarts 
   * autosleep-app/0 start:
      * cf push --no-start autosleep-v105-i1 -p /var/vcap/jobs/../autosleep.jar
      * cf bind-service autosleep-v105-i1 autosleep-mysql
      * FAILED because autosleep-mysql does not exist 

How to reuse the release across apps ?
* a bosh-gen like template
* a generic bosh release (git submodule) 

# FAQ

# Why use bosh to deploy apps on CF ?

Bosh is attractive for platform teams because:
* well known tool: it is the main framework/toolchain used to deploy CF and services
* helps transition from build to run:
  * properties are documented
  * releases are versionned (dev and final)
  * final releases are self-contained and do not require external dependencies
  * canary releases update

CF is attractive for a number of platform components w.r.t. to plain bosh-based packaging:
- domains and routes
- centralized logs
- app runtime available out of the box through buildpacks
- security groups
- quotas
- ...

 
