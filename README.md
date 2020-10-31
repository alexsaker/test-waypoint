# Test Waypoint

## Resources
|Title|Description|
|---|---|
|Docs|[Azure ACI](https://www.waypointproject.io/plugins/azure-container-instances)|
|Tutorial|[HashiCorp Learn](https://learn.hashicorp.com/tutorials/waypoint/azure-container-instance?in=waypoint/deploy-azure)|



## Install waypoint cli
```bash
url -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install waypoint
```

## Login to Azure
```bash
az login
```

## Create a container registry and update it to be admin enabled
```bash
az acr update -n <registryName> --admin-enabled true
az acr login --name <registryName>
```

## Set Azure credentials as env variable
Check out values in your azure portal or using your cli/powershell
```bash
export AZURE_SUBSCRIPTION_ID="<subscription id>"
export REGISTRY_USERNAME="<username>"
export REGISTRY_PASSWORD="<password>"
```


## Install local waypoint server
```bash
docker pull hashicorp/waypoint:latest
waypoint install --platform=docker -accept-tos
```

## Configure waypoint hcl
```groovy
project = "example-nodejs"

app "example-nodejs" {
  labels = {
    "service" = "example-nodejs",
    "env"     = "dev"
  }

  build {
    use "pack" {}

    registry {
      use "docker" {
        image = "<registryName>.azurecr.io/example-nodejs"
        tag   = "latest"
      }
    }
  }

  deploy {
    use "azure-container-instance" {
      resource_group = "<resourceNAme>"
      location       = "francecentral"
      ports          = [8080]

      capacity {
        memory    = "512"
        cpu_count = 1
      }

    }
  }

}
```

## Initialize and run waypoint project
```bash
waypoint init
waypoint up
```
If displyed url is not good, you can refer to the logs to point you to the error.
Otherwise you can check the url on the container instances page @ https://portal.Azure.com


## clean up
 ```bash
 waypoint destroy
 az group delete -n waypoint-test
 ```