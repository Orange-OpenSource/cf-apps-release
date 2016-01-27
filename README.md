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
# Version upgrade with 20% canary=1
# app version update
# service update
# Version rollback
# Configuration change
# Scale out
# Snapshot
$ bosh take snapshot
# delete app
# 

```

# FAQ

# Why use bosh to deploy apps on CF ?

Bosh is attractive for platform teams because:
* well known tool: it is the main framework/toolchain used to deploy CF and services
* helps transition from build to run:
  * properties are documented
  * releases are versionned (dev and final)
  * final releases are self-contained and do not require external dependencies

CF is attractive for a number of platform components w.r.t. to plain bosh-based packaging:
- domains and routes
- centralized logs
- app runtime available out of the box through buildpacks
- security groups
- quotas
- ...

 
