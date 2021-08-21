# Bootstrap

The bootstrap script creates/initializes the authentication
and base resources needed.

For an Azure deployment, this involves creating/initializing 3 things -

1. Subscription ID (extracted from the account or specified by the user)
2. Azure Active Directory Application
3. Service Principal

While ideally, these would be created/managed by terraform,
it requires these to be pre-created. So we'll use a shell script to get these configured. 

## Subscription ID

As we explored in the section on Azure Management Hierarchies, each Azure deployment is contained within a "subscription". Subscriptions are a logical partition, allowing billing and management control. Creating a subscription is beyond the scope of this document, however, by having a personal login to the Azure cloud, you are working within the purview of a "subscription". At a minimum, it has a clearly defined party to be billed for resources consumed. This could be an employer or your personal credit card used to setup the Azure account.

While we do not create it, we do need the subscription id to create a Resource Group, which will in turn contain all the resources we consume.

The subscription ID can vary by login. While it's possible (even preferable) to hard code this value for a team, when running this script on your local machine, this will likely vary.

The bootstrap script automates it's extraction - after you have logged via the azure CLI.

<!-- tabs:start -->
### **windows**

(win/support/bootstrap.bat)

```batch win/support/bootstrap.bat

echo "\n*****************"
echo   "* Bootstrapping *"
echo   "*****************\n\n"

if (%SUBSCRIPTION_ID% == '') (
  echo "Extracting Azure 'Subscription ID' from current login"
  rem Opens a webpage to login to Azure and provides credentials to the azure-cli
  az login

  rem Picks the subscription tied to the login above
  rem az account show --query 'id' --output tsv
  FOR /F "tokens=* USEBACKQ" %%g IN (`az account show --query 'id' --output tsv`) do (SET SUBSCRIPTION_ID=%%g)
)

if (%SUBSCRIPTION_ID% == '') (
  echo "%RED%ERROR: Subscription ID not found%NC%"
  exit /b -1
) else (
  echo "%YELLOW%SUBSCRIPTION_ID:%CYAN% %SUBSCRIPTION_ID% %NC%"
)
```

### **\*nix**
(nix/support/bootstrap.sh)

```sh nix/support/bootstrap.sh

echo "\n*****************"
echo   "* Bootstrapping *"
echo   "*****************\n\n"

# specify SUBSCRIPTION_ID here if you'd like to pin it to a specific one.
# by default, will ask you to login and use the SUBSCRIPTION_ID tied to your account
if [[ ${SUBSCRIPTION_ID} == '' ]]
then
  echo "Extracting Azure 'Subscription ID' from current login"

  # Opens a webpage to login to Azure and provides credentials to the azure-cli
  az login

  # Picks the subscription tied to the login above
  # az account show --query 'id' --output tsv
  SUBSCRIPTION_ID=$(az account show --query 'id' --output tsv)
fi

SUBSCRIPTION_NAME=$(az account show --subscription ${SUBSCRIPTION_ID} --query 'name')

if [[ ${SUBSCRIPTION_ID} == '' ]]
then
 echo "${RED}ERROR: Subscription ID not found${NC}"
 exit -1
else
 echo "${YELLOW}SUBSCRIPTION_ID:${CYAN} ${SUBSCRIPTION_ID} ${NC}"
 echo "${YELLOW}SUBSCRIPTION_NAME:${CYAN} ${SUBSCRIPTION_NAME} ${NC}"
fi
```
<!-- tabs:end -->

## Resource Group

### globals
Set resource names in globals.{sh|bat}, allowing them
to be used by other scripts/lifecycle-methods

<!-- tabs:start -->
#### **windows**
(globals.bat)

```batch globals.bat
set RESOURCE_GROUP=%APP_NAME%-rg
```

#### **\*nix**
(globals.sh)

```sh globals.sh
RESOURCE_GROUP=${APP_NAME}-rg
```
<!-- tabs:end -->

If not already existing, create an Azure resource-groupm for all Azure resource
to be created.

<!-- tabs:start -->
### **windows**

(win/support/bootstrap.bat)

```batch win/support/bootstrap.bat
set RESOURCE_GROUP=%APP_NAME%-rg

echo "\nResource Group '%RESOURCE_GROUP%'"

FOR /F "tokens=* USEBACKQ" %%g IN (`az group exists -n %RESOURCE_GROUP%`) do (SET rgExists=%%g)


if (%rgExists%=='true') (
  echo "Reusing existing resource group '${RESOURCE_GROUP}'"
) else (
  if (%USE_CASE%=='create') (
    echo "Creating resource-group '%RESOURCE_GROUP%'"
    az group create \
      --name %RESOURCE_GROUP% \
      --location %REGION_NAME%
  ) else (
    echo "%RED% ERROR: Can only create ResourceGroup with USE_CASE=create, not %USE_CASE%%NC%"
    exit /b -1
  )
)
FOR /F "tokens=* USEBACKQ" %%g IN (`az group show --query 'id' -n %RESOURCE_GROUP%`) do (SET RESOURCE_GROUP_ID=%%g)

if (%RESOURCE_GROUP_ID% == '') (
  echo "%RED%ERROR: RESOURCE_GROUP_ID not found%NC%"
  exit /b -1
) else (
  echo "%YELLOW%RESOURCE_GROUP_ID:%CYAN% %RESOURCE_GROUP_ID% %NC%"
)
```

### **\*nix**

(nix/support/bootstrap.sh)

```sh nix/support/bootstrap.sh
echo "\nResource Group '${RESOURCE_GROUP}'"

rgExists=$(az group exists -n ${RESOURCE_GROUP})

if [[ ${rgExists} == 'true' ]];
then
  echo "Reusing existing resource group '${RESOURCE_GROUP}'"
elif [[ "${USE_CASE}" == "create" ]]
then
  echo "Creating resource-group '${RESOURCE_GROUP}' ${REGION_NAME}"
  # echo "az group create --name ${RESOURCE_GROUP} --location ${REGION_NAME}"
  # trap return value of next command & discard. bash seems to treat it as an error.
  JUNK=$(az group create --name ${RESOURCE_GROUP} --location ${REGION_NAME})
  unset JUNK
else
  echo "${RED} ERROR: Can only create ResourceGroup with USE_CASE=create, not ${USE_CASE}${NC}"
  exit -1
fi

RESOURCE_GROUP_ID=$(az group show --query 'id' -n ${RESOURCE_GROUP} -o tsv)

if [[ ${RESOURCE_GROUP_ID} == '' ]]
then
 echo "${RED}ERROR: RESOURCE_GROUP_ID not found${NC}"
 exit /b -1
else
 echo "${YELLOW}RESOURCE_GROUP_ID:${CYAN} ${RESOURCE_GROUP_ID} ${NC}"
fi

```
<!-- tabs:end -->


## Azure Active Directory Application

We need a service principal to generate credentials for automation to access necessary resources.
However, before you create a service principal, you need to create an “application” in Azure Active Directory. You can think of this as an identity for the application that needs access to your Azure resources.

<!-- tabs:start -->

###  **windows**
(win/support/bootstrap.bat)

```batch win/support/bootstrap.bat

echo "\nActive Directory Application '%APP_NAME%'"

rem fetch previously created app
FOR /F "tokens=* USEBACKQ" %%g IN (`az ad app list --query '[].appId' -o tsv --display-name %APP_NAME%`) do (SET APP_ID=%%g)

if (%APP_ID%=="") (
  if (%USE_CASE%=='create') (
    rem APP_ID not found, create new Active directory app
    az ad app create --display-name %APP_NAME%
    echo "created new App '%APP_NAME%'"
  ) else (
      echo "ERROR: Can only create App with USE_CASE=create, not %USE_CASE%"
      exit /b -1
  )
)

rem extract APP_ID (needed to create the service principal)
FOR /F "tokens=* USEBACKQ" %%g IN (`az ad app list --query '[].appId' -o tsv --display-name %APP_NAME%`) do (SET APP_ID=%%g)

if (%APP_ID% == '') (
  echo "%RED%ERROR: APP_ID not found%NC%"
  exit /b -1
) else (
  echo "%YELLOW%APP_ID:%CYAN% %APP_ID% %NC%"
)
```

### **\*nix**
(nix/support/bootstrap.sh)

```sh nix/support/bootstrap.sh

echo "\nActive Directory Application '${APP_NAME}'"

appExists=$(az ad app list --query '[].appId' -o tsv --display-name ${APP_NAME})

# fetch previously created app
if [[ ${appExists} != "" ]]
then
  echo "Reusing AD Application '${APP_NAME}'"
else
  # APP_ID not found, create new Active directory app
  _APP=$(az ad app create --display-name ${APP_NAME})
  echo "Created AD Application '${APP_NAME}'"
fi

# extract APP_ID (needed to create the service principal)
APP_ID=$(az ad app list --query '[].appId' -o tsv --display-name ${APP_NAME})

if [[ ${APP_ID} == '' ]]
then
 echo "${RED}ERROR: APP_ID not found${NC}"
 exit -1
else
 echo "${YELLOW}APP_ID:${CYAN} ${APP_ID} ${NC}"
fi

```
<!-- tabs:end -->



## Service Principal

With subscription id, resource-group and AD application in hand, we are now in a position to create our service principal.

A [service principal](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2Fazure%2Fazure-resource-manager%2Ftoc.json&view=azure-cli-latest) provides authentication for automation/non-human entities. We can assign a "role" with specific access privileges or "Role based access control (RBAC)".

Creation of the service principal returns the authenticatin credentials. We will sore these to a file, allowing us to use this to setup secrets for github actions.

> NOTE: It seems managed identities might be a better way to provide RBAC privileges. However we are unclear as to the final set of resources needed and whether all resources required will support Managed identities. A move to managed identities will need to be investigated and done at a later point in time.

### globals
Extract resource names into globals.{sh|bat} to allow use in other lifecycle methods

<!-- tabs:start -->

#### **windows**

(globals.bat)

```batch globals.bat
set SERVICE_PRINCIPAL=%APP_NAME%-sp
set SP_FNAME=./%SERVICE_PRINCIPAL%-creds.dat
```

#### **\*nix**

(globals.sh)

```sh globals.sh
SERVICE_PRINCIPAL=${APP_NAME}-sp
SP_FNAME=./${SERVICE_PRINCIPAL}-creds.dat
```

<!-- tabs:end -->


<!-- tabs:start -->

### **windows**

(win/support/bootstrap.bat)

```batch win/support/bootstrap.bat

rem create service principal
echo "\nService Principal '%SERVICE_PRINCIPAL%'"

if exist %SP_FNAME% (
  echo "Reusing existing service principal %APP_NAME%-sp"
  node %SCRIPT_ROOT%/bin/decrypt.js %SP_FNAME%
  FOR /F "tokens=* USEBACKQ" %%g IN (`type %SP_FNAME%.decrypted`) do (SET SP_CREDENTIALS=%%g)
  call %SP_FNAME%.env
  del %SP_FNAME%.env
  del %SP_FNAME%.decrypted
) else (
  if( %USE_CASE% == 'create) (
    FOR /F "tokens=* USEBACKQ" %%g IN (
      $(az ad sp create-for-rbac \
        --sdk-auth \
        --skip-assignment \
        --name %SERVICE_PRINCIPAL%`) do (SET SP_CREDENTIALS=%%g)
    echo %SP_CREDENTIALS% > %SP_FNAME%
    node %SCRIPT_ROOT%/bin/encrypt.js %SP_FNAME%
    call %SP_FNAME%.env
    del %SP_FNAME%.env
  ) else (
    echo "ERROR: Cannot only create ServicePrincipal with USE_CASE='create', not %USE_CASE%"
    exit /b -1
  )
)
FOR /F "tokens=* USEBACKQ" %%g IN (`az ad sp list --query '[].objectId' -o tsv --display-name %SERVICE_PRINCIPAL%`) do (SET SERVICE_PRINCIPAL_ID=%%g)

if (%SERVICE_PRINCIPAL_ID% == '') (
  echo "%RED%ERROR: SERVICE_PRINCIPAL_ID not found%NC%"
  exit /b -1
) else (
  echo "%YELLOW%SERVICE_PRINCIPAL_ID:%CYAN% %SERVICE_PRINCIPAL_ID% %NC%"
)

```

### **\*nix**

(nix/support/bootstrap.sh)

```sh nix/support/bootstrap.sh

# create service principal and store/use encrypted credentials
echo "\n${YELLOW}SERVICE_PRINCIPAL: ${CYAN}${SERVICE_PRINCIPAL}${NC}"

if test -f ${SP_FNAME}
then
  # use previously encrypted credentials. Will prompt for 'password'
  node ${SCRIPT_ROOT}/bin/decrypt.js ${SP_FNAME}
  SP_CREDENTIALS=`cat ${SP_FNAME}.decrypted`
  . ${SP_FNAME}.env
  rm ${SP_FNAME}.env
  rm ${SP_FNAME}.decrypted
else
  SP_CREDENTIALS=$(az ad sp create-for-rbac \
    --sdk-auth \
    --name ${SERVICE_PRINCIPAL})
  echo ${SP_CREDENTIALS} > ${SP_FNAME}
  # encrypt secrets with 'password' for next iteration.
  node ${SCRIPT_ROOT}/bin/encrypt.js ${SP_FNAME}
  . ${SP_FNAME}.env
  rm ${SP_FNAME}.env
fi

SERVICE_PRINCIPAL_ID=$(az ad sp list --query '[].objectId' -o tsv --display-name ${SERVICE_PRINCIPAL})

if [[ ${SERVICE_PRINCIPAL}_ID == '' ]]
then
 echo "${RED}ERROR: SERVICE_PRINCIPAL_ID not found${NC}"
 echo "(in development, try deleting the credentials file '${SP_FNAME}' and retry)"
 exit /b -1
else
 echo "${YELLOW}SERVICE_PRINCIPAL_ID:${CYAN} ${SERVICE_PRINCIPAL}_ID ${NC}"
fi

```
<!-- tabs:end -->


**Footnote**

Adds run-time visual confirmation of success to the user. 

<!-- tabs:start -->

#### **windows**

(win/support/bootstrap.bat)

```bat win/support/bootstrap.bat
echo "\n\n%GREEN%Bootstrap successful%NC%"
```

#### **\*nix**

(nix/support/bootstrap.sh)

```sh nix/support/bootstrap.sh
echo "\n\n${GREEN}Bootstrap successful${NC}"
```

<!-- tabs:end -->
