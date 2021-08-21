# Globals

A cloud orchestration is necessarily involves multiple
tools, which need an ability to communicate resource/object
names. We use environment variables specified in `globals.{bat|sh}`
as a mechanism for this.

We start with the minimal configuration here, with 6 variables that
the user needs to provide:

<!-- tabs:start -->

## **windows**
(globals.bat)

```batch globals.bat
rem github organization
set GH_ORG=${GH_ORG:-acuity-sr}

rem github repository 
set GH_REPO=${GH_REPO:-acuity-bkstg}

rem github release/tag to be deployed
RELEASE=${RELEASE?}

rem github branch name, used as 'stage' name of deployment
STAGE=${STAGE:?'main'}

rem azure region for deployment - all resources are created here.
REGION_NAME=${REGION_NAME:?'eastus'}

rem the subscription id to be used to create the account
set SUBSCRIPTION_ID=d8f43804-1ed0-4f0d-b26d-77e8e11e86fd

rem ----
rem variables below this point are derived from those above
rem modifications is typically not necessary

rem APP_NAME is used as the root-name for all resources created
rem typically used with resources that might conflict
set APP_NAME=%GH_REPO%-%STAGE%-%REGION%

rem for when we need a short form - like within a k8s cluster, 
rem where name-collision has limited scope.
set APP=%GH_REPO%

pushd %~dp0
set SCRIPT_ROOT=%CD%
popd

rem the project src is typically one level up from the dev-ops "scripts" folder.
SRC_DIR=%SCRIPT_ROOT%/..

rem convenience definitions
set RED="[31m [31m"
set GREEN="[32m [32m"
set YELLOW="[33m [33m"
set BLUE="[34m [34m"
set PURPLE="[35m [35m"
set CYAN="[36m [36m"
rem NO_COLOR
set NC="[0m"

rem ----
rem variables below this point are auto-generated, modifications are ill-advised
```

## **\*nix**
(globals.sh)

```bash globals.sh
# github organization
GH_ORG=${GH_ORG:-acuity-sr}

# github repository 
GH_REPO=${GH_REPO:-acuity-bkstg}

# github release/tag to be deployed
RELEASE=${RELEASE?}

# github branch name, used as 'stage' name of deployment
STAGE=${STAGE:?'main'}

# azure region for deployment - all resources are created here.
REGION_NAME=${REGION_NAME:?'eastus'}

# the subscription id to be used to create the account
SUBSCRIPTION_ID=d8f43804-1ed0-4f0d-b26d-77e8e11e86fd


# ----
# variables below this point are derived from those above
# modifications is typically not necessary

# APP_NAME is used as the root-name for all resources created
# typically used with resources that might conflict
APP_NAME=%GH_REPO%-%STAGE%-%REGION%

# for when we need a short form - like within a k8s cluster, 
# where name-collision has limited scope.
APP=%GH_REPO%

# debug controls
set -e # stop on error
# set -x # echo commands, expand variables
# set -v # echo commands, do not expand variables

SCRIPT_ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"

# the project src is typically one level up from the dev-ops "scripts" folder.
SRC_DIR=${SCRIPT_ROOT}/..

# convenience definitions
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
CYAN='\033[0;36m'
# NO_COLOR
NC="\033[0m"

# ----
# variables below this point are auto-generated, modifications are ill-advised
```
<!-- tabs:end -->