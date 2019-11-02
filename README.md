# baseimage-node

## Fork the repo for your own changes

https://github.com/demo42/baseimage-node

## Setup defaults

```sh
export DEMO_NAME=demo42
export ACR_NAME=demo42t
export AKV_NAME=demo42
export LOCATION=southcentralus
export RESOURCE_GROUP=$DEMO_NAME
export BASE_IMAGE_NODE_REPO=https://github.com/demo42/baseimage-node.git
export GIT_TOKEN_NAME=${DEMO_NAME}-git-token

az configure --defaults acr=$ACR_NAME
```

## Create ACR

```sh
az acr create -n $ACR_NAME -l $LOCATION -g $RESOURCE_GROUP --sku standard
```

## Create a Personal Access Token

- See [ACR Build Docs](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-build-task#create-a-github-personal-access-token) for specific permissions
- Copy the PAT

## Azure KeyVault

- Create the KeyVault

    ```sh
    az keyvault create --resource-group $RESOURCE_GROUP --name $AKV_NAME
    ```

- Save the Github Personal Access Token

    ```sh
    az keyvault secret set \
        --vault-name $AKV_NAME \
        --name $GIT_TOKEN_NAME \
        --value <past-PAT-value>
    ```

- Verify the values were saved

    ```sh
    az keyvault secret show \
                         --vault-name $AKV_NAME \
                         --name $GIT_TOKEN_NAME \
                         --query value -o tsv
    ```

## Create the Task

```sh
az acr task create \
  -n base-image-node \
  -f acr-task.yaml \
  --context $BASE_IMAGE_NODE_REPO \
  --git-access-token $(az keyvault secret show \
                         --vault-name $AKV_NAME \
                         --name $GIT_TOKEN_NAME \
                         --query value -o tsv)
```

## Manually Trigger a Build

```sh
az acr build-task run -n base-image-node
```
