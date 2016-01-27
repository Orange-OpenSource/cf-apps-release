# cf-apps-release
A generic bosh release to deploy apps on CF 

# FAQ

# Why use bosh to deploy apps on CF ?

Bosh is attractive for platform teams because:
* well known tool: it is the main framework/toolchain used to deploy CF and services
* helps transition from build to run:
  * properties are documented
  * releases are versionned (dev and final)
  * final releases are self-contained and do not require external dependencies

CF is attractive for a number of platform components w.r.t. to plain bosh-based packaging:
- centralized logs
- app runtime available out of the box through buildpacks
- quotas
- ...

 
