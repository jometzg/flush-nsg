# Flush Connections on a Network Security Group
REST example of how to flush an Network Security Group (NSG) rules to allow for immediate re-evaluation

## Rationale
Network Security Group (NSG) rules are executed against *new* network flows. If there is an on-going network flow and a new *Deny* rule is added, then this will not have an affect on the network flow.
In some circumstances, it is useful to flush the connection to allow the rule to be re-evaluated.

## Solution
There is a [REST interface](https://learn.microsoft.com/en-us/rest/api/virtualnetwork/network-security-groups/create-or-update?tabs=HTTP) that allows the update of an NSG. This may be used to update the NSG.
This sample uses the [Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) from Visual Studio Code.

### Authenticate
This uses an Azure service principal that has enough permissions to do the job.
```
az ad sp create-for-rbac --scopes /subscriptions/your-subscription-id  --name "sp-name" --role "Contributor"
```
This returns a clientId and secret. These can be used below

```
### variables for my personal subscription  - you will need to change these
@directoryid = your-tenant-id
@clientid = your-service-principal-client-id
@secret = your-service-principal-client-secret
@subscriptionid = your-subscription-id

### get a bearer token
# @name login
POST https://login.microsoftonline.com/{{directoryid}}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={{clientid}}
&scope=https://management.core.windows.net/.default
&client_secret={{secret}}
&grant_type=client_credentials

### grab the access token
@authToken = {{login.response.body.access_token}}
```

The variable *authToken* should contain the bearer token

### List the Network Secrurity Groups in your subscription
```
### get list of NSGs for this subscription
GET https://management.azure.com/subscriptions/{{subscriptionid}}/providers/Microsoft.Network/networkSecurityGroups?api-version=2022-11-01
Authorization: Bearer {{authToken}}

```
### Get the details of a specific NSG
```
### set these for later queries
@resourceGroupName = your-nsg-resource-group
@networkSecurityGroupName = your-nsg-name
@location = westeurope

### get a specific NSG
GET https://management.azure.com/subscriptions/{{subscriptionid}}/resourceGroups/{{resourceGroupName}}/providers/Microsoft.Network/networkSecurityGroups/{{networkSecurityGroupName}}?api-version=2022-11-01
Authorization: Bearer {{authToken}}
```

### Update a specific Network Security Group
```
### update a specic NSG to flush connections
PUT https://management.azure.com/subscriptions/{{subscriptionid}}/resourceGroups/{{resourceGroupName}}/providers/Microsoft.Network/networkSecurityGroups/{{networkSecurityGroupName}}?api-version=2022-11-01
Authorization: Bearer {{authToken}}
Content-Type: application/json

{
    "id": "/subscriptions/{{subscriptionid}}/resourceGroups/{{resourceGroupName}}/providers/Microsoft.Network/networkSecurityGroups/{{networkSecurityGroupName}}",
    "location": "{{location}}",
    "properties": {
        "flushConnection": true
    }
}
```
The response from this should contain "provisioningState": "Succeeded".


### The REST client code
This is contained in the file *rest-flush-nsg-connection*.
