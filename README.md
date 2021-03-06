# *CAD Customer Recommendations Business Scenario Lab*

Note, this Lab can be used as an extension to the Personalisation lab which may be found here 
[Personalisation](https://github.com/shanepeckham/CADScenario_Personalisation). This will showcase a comprehensive and rich end to end scenario.

# What is it?

This repository will provision an environment that may be used as a Lab to build an end to end scenario that does the following:

*	Get the recommended products for a user based on product purchasing patterns. A sample dataset is provided but a custom one can be loaded too.
*	Perform automated transformation 
*	Prepare the output for a legacy website that is on-premise

# What does it showcase?

This solution brings together Infrastructure as a Service (IaaS), Platform as a Service (PaaS), Software as a Service (SaaS), Container as a Service (CaaS) and Serverless and Cognitive Services components on Microsoft Azure to build a realistic end to end scenario to generate recommendations for a customer based on other product purchase patterns using the Recommendations API. This solution will show how an on-Premise XML File based legacy application can be quickly modernised using a microservice container based API for a front end. Without needing to change any of the backend code, we will provide a modern microservice REST and JSON based API to front the legacy app that can serve a mobile app.

# The end to end scenario

This solution will determine the list of recommended products for a particular customer and return the response in JSON. A synchronous process will convert the message to XML and write it back to an On-Premise legacy XML File system. The essence of this lab is to show how a legacy application can provide a modern user experience (in this case serving recommendations to a mobile app) wihtout needing to refactor the legacy application. 

# The solution aims to show the following:

*	How legacy lift and shift applications on IaaS can be incorporated into modern solutions to quickly derive value from higher value services in the cloud.
*	How existing investments can be modernized without having to rebuild everything to drive customer value
*	The ease with which On-premise, public and private components may be brought together to build workloads that bring business value
*	The meshing of IaaS, PaaS, SaaS, Serverless and AI with tools that are accessible to non-developers
*	OSS workloads running on Azure

# Technology used

The following technology components are used in this solution:

*	Swagger enabled Node.js APIs running in a container on the Azure Linux App Services (PaaS) (CaaS)
*   The Azure Container Registry to privately and securely store approved container images
*	A legacy Windows machine running on-Premise or on a VM that serves as an XML File system for a conceptual legacy website.
*	Azure networking to isolate legacy workloads (IaaS)
*	API Management to govern APIs and to perform automated JSON<-->XML conversion and amend requests inflight
*	Azure Logic apps to provide serverless integration that is accessible to non-developers (Serveless)
* The Logic apps on-Premise Data Gateway to connect to an on-Premise machine or a VM in an isolated VNET
*	Azure Resource Manager templates to automate the provisioning and inflation of a full environment

# Solution flow

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/topology2.png "Solution Flow")

# The Lab component

This solution will install most of the components required to build the end to end Recommendations scenario. The Lab attendees need to configure a few components, install the on-Premise Data Gateway, prepare an API Management Policy and wire everything together in a Logic App. 

# Preparing for the solution

For this Lab you will require (note, these items can be all be installed on the VM that will be provisioned during the lab):

* Install Postman, get it here - https://www.getpostman.com
* Install Docker, get it here - https://docs.docker.com/engine/installation/
* Install the Logic App On-Premise Data Gateway, get it here - https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install

# How to install the solution

## 1. Provisioning the components: Select Deploy to Azure to deploy to your Azure instance that you are currently logged in to.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fshanepeckham%2FCADScenario_Recommendations%2Fmaster%2Fazuredeployrecommendations.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fshanepeckham%2FCADScenario_Recommendations%2Fmaster%2Fazuredeployrecommendations.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

Select or create a Resource Group to deploy to and the only parameter you need to change is the Deployment Name - give it any name of 12 characters or less as it will be used to generate a hash to ensure your site names are unique, see the image below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/DeploymentName.png)

This will take roughly 20-30 minutes as this will provision:

* One VNET
* A Windows VM and place in inside the VNET isolated with NSGs
* An API Management instance (Developer Tier)
* An Azure Container Service Registry
* A Linux App Service API app 
* Storage accounts to house the VM VHD, the App Service API logging and the Azure Container Registry
* A Cognitive Services account configured for the Recommendations API

## 2. Installing the on-Premise Data Gateway for Logic Apps

Navigate to your VM, the default name will be LegacyXML[hash] and navigate to the Overview blade and copy the value in the field Public IP Address/DNS label, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/legacyip.png)

Connect to the machine via Remote Desktop with the following credentials:
User: MiniCADAdmin
Password: MiniCADAdmin123

***Take note of the value in the domain as you may need to use this later

Navigate to https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install and download the Data Gateway.

Click next on the first step of the install, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg1.png)

Accept the default install location and accept the terms and conditions, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg2.png)

You should now get to a screen that states that the install was successful, now you need to enter an email address to identify the gateway account, use the email address associated with your Azure subscription and click 'Sign in', see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg3.png)

Complete the logon process to Azure.

Now you need to enter the Gatewat Server Name and a recovery key, see below:

Enter *LegacyXML* as the Gateway Name and select a recovery key that you won't forget and copy it somewhere just in case. Ensure that the region highlighted below is the same as the region that you selected to deploy the components to from Step 1 above, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg4.png)

The image below shows the region change option:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg5.png)

You should get a success screen as below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg6.png)

### Now we need to create a share that will hold the XML Files for the legacy application

Open up Windows Explorer and create a new folder on the C:\ drive called *ShoppingXML*, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg7.png)

Now right click the ShoppingXML directory and select Properties --> Sharing, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg8.png)

Select 'Share' and highlight the *MiniCADAdmin* user to share with and click the 'Share' button on the bottom right of the popup, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg9.png)

You should now see that the folder has been shared, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg10.png)

### Now we need to install the On-Premise Data Gateway component within the Azure subscription

Select New in the Azure portal and enter *Data Gateway* and select the entry for 'On-premises Data Gateway', see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg11.png)
![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg12.png)

You will be presented with the configuration screen for the 'On-premises Data Gateway', ensure that you select the same *Resource Group* and *Location* that selected for the initial deploy and install, you should now see your Gateway appear in the *Installation Name* field, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg13.png)

## 3. Now we will set up the Azure Container Registry

Navigate to Container Registry component provisioned within Azure, its name will be generated by default with the following format RecommACS[hash].

Click on the *Quick Start* blade, this will provide you with the relevant commands to upload a container image to your registry, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/quicksstartacs.png)

Now start a command/terminal window, create a directory called CADRecommendations and type the following:

``` 
docker pull shanepeckham/cadrecommendations
```
This will pull down the container image to your local machine that we will use for our API later on.

You should see a status similar to below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dockerpullcad.png)

Now we will push the image up to the Azure Container Registry, enter the following (from the quickstart screen):

``` 
docker login recommacs[hash].azurecr.io

```

To get the username and password, navigate to the *Access Keys* blade, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/acskeys.png)

You will receive a 'Login Succeeded' message. Now type the following:
```
docker tag shanepeckham/cadrecommendations recommacs[hash].azurecr.io/cadrecommendations
docker push recommacs[hash].azurecr.io/cadrecommendations
```
Once this has completed, you will be able to see your container uploaded to the Container Registry within the portal, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/repos.png)

## 4. Train the Recommendations Model

Note, you can either train this model on your own Recommendations historic data or you can use the data in this lab, which is an extension of the data generated by the Bot in the [Personalisation](https://github.com/shanepeckham/CADScenario_Personalisation) lab.

More [info on the Recommendations API](https://westus.dev.cognitive.microsoft.com/docs/services/Recommendations.V4.0/) can be found here on all the methods: 

Navigate to Recommendations Cognitive Services component that was provisioned within Azure, its name will be generated by default with the following format RecommendationsCS[hash].

Click on the *Keys* blade and copy the value from one of the keys, this is an API Management authorisation key and will be required for every interaction, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/cskeys.png)

Now open Postman as we will use the REST API to train our model.

### Creating the model

Create a new request with the following parameters:

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models*
Request headers: 
        Content-Type: application/json
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
{
  "modelName": "CADRecommendations",
  "description": "CADRecommendations"
}
```
This will return the id for the model created, store this value, we will refer to it as modelId in the URI exmaples below.

### Upload the catalogue

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/catalog?catalogDisplayName=coffeecatalogue*
Request headers: 
        Content-Type: text
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
1, Latte, White
2, Americano, Black
3, Cappucino, White
4, Espresso, Black

```

We will use the contents of the [coffeecatalogue](https://raw.githubusercontent.com/shanepeckham/CADScenario_Recommendations/master/coffeecatalogue.txt) lab.
 file if you do not have specific catalogue to train.

### Upload the usage file

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/usage?usageDisplayName=coffeeusage*
Request headers: 
        Content-Type: text
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
2,2,2018-02-03 07:01:34
4,4,2017-10-14 13:57:27
4,1,2018-03-07 03:03:19
4,2,2017-01-10 21:05:20
1,1,2017-11-08 04:20:41
2,4,2017-03-30 08:07:27
1,3,2018-02-02 01:53:12
1,3,2017-04-10 06:08:44
1,2,2017-09-16 03:47:14
2,1,2016-12-13 13:12:45
3,3,2016-11-16 12:58:10
3,3,2017-03-24 17:21:38
3,3,2018-01-11 22:36:38
4,2,2017-01-19 20:54:27
4,1,2017-08-31 00:11:38
4,2,2016-12-05 12:25:08
2,1,2017-10-24 08:04:05
1,4,2016-12-13 00:11:22
2,2,2017-04-16 03:10:20
4,3,2017-07-13 03:47:20
3,3,2017-07-19 21:47:35
2,4,2016-11-08 19:22:37
2,3,2017-08-18 07:04:34
4,1,2018-02-19 15:35:53
2,4,2016-06-13 02:21:08
1,1,2017-09-01 20:00:08
1,1,2017-10-07 16:06:53
4,1,2017-04-11 08:05:58
4,4,2017-11-06 08:23:54
3,1,2017-10-26 17:57:24
3,3,2018-01-28 05:45:34
2,4,2017-07-07 12:38:42
3,2,2017-10-31 13:36:21
2,2,2016-06-16 08:39:35
2,2,2017-11-04 10:49:04
2,4,2017-11-11 19:06:58
4,3,2017-07-31 22:30:41
1,1,2017-01-13 16:07:23
1,1,2017-07-02 09:46:34
3,1,2017-06-15 16:20:18
4,2,2017-09-12 04:27:33
3,4,2016-07-01 19:17:01
3,4,2016-12-27 12:06:11
3,1,2017-01-11 04:44:14
4,3,2016-10-01 21:17:57
2,4,2017-03-14 00:54:48
4,1,2017-11-25 00:55:41
3,1,2016-08-28 11:29:28
2,4,2017-08-31 20:18:22
1,1,2016-06-07 18:46:15
2,2,2017-08-31 23:36:11
1,1,2017-05-28 00:08:37
2,1,2017-11-30 11:59:13
2,4,2018-02-10 09:01:37
2,1,2017-06-13 03:58:46
3,3,2018-03-26 15:55:39
1,1,2017-02-19 22:31:32
3,1,2016-10-09 17:03:38
3,4,2017-02-02 16:07:26
2,1,2017-03-15 02:10:54
2,3,2017-04-17 14:56:40
4,3,2016-07-10 02:05:46
2,1,2017-06-03 10:46:05
1,4,2017-02-01 05:10:36
1,1,2018-03-11 09:15:40
1,2,2017-01-20 11:55:16
3,1,2017-02-27 12:08:57
4,4,2016-10-23 07:24:00
3,1,2017-01-11 08:50:29
3,2,2016-10-29 18:28:02
1,3,2016-11-26 11:29:49
4,1,2017-01-02 19:26:17
2,3,2017-03-28 16:14:57
1,4,2017-09-12 00:19:02
1,3,2017-06-07 08:53:16
4,2,2017-02-08 03:18:41
4,4,2018-03-10 11:12:35
3,1,2017-05-11 18:35:32
4,2,2017-03-21 12:43:57
1,1,2016-10-14 05:23:42
4,2,2017-06-01 16:36:46
2,2,2017-05-06 05:28:17
1,1,2016-05-03 18:07:50
4,1,2018-04-17 12:12:05
2,2,2016-10-21 13:04:10
1,3,2016-06-18 21:01:02
1,1,2017-08-14 17:00:27
2,4,2017-11-08 18:20:58
1,4,2016-09-17 05:06:48
1,1,2016-12-31 10:01:16
3,1,2016-08-06 23:35:42
2,1,2017-12-01 15:03:38
1,3,2017-07-11 08:09:07
2,2,2016-09-17 01:23:41
1,2,2017-11-06 12:17:19
1,1,2016-06-29 23:24:06
1,4,2017-12-01 12:55:09
2,2,2018-01-05 17:15:14
4,2,2016-11-22 18:30:00
3,4,2016-12-02 09:57:41

```

We will use the contents of the [coffeecatalogue](https://raw.githubusercontent.com/shanepeckham/CADScenario_Recommendations/master/coffeeusage.txt) lab.
 file if you do not have specific catalogue to train.

 ### Create the Trigger/Build the model
 
We will train the model to provide item to item recommendations, in other words, based on that product sold, which other product is the user/customer likely to purchase.

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/builds*
Request headers: 
        Content-Type: application/json
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
{

"description": "CAD: Simple recomendations build",

"buildType": "recommendation",

"buildParameters": {

    "recommendation": {

      "numberOfModelIterations": 20,

      "numberOfModelDimensions": 20,

      "itemCutOffLowerBound": 1,

      "itemCutOffUpperBound": 10,

      "userCutOffLowerBound": 0,

      "userCutOffUpperBound": 0,

      "enableModelingInsights": false,

      "useFeaturesInModel": false,

      "modelingFeatureList": "string",

      "allowColdItemPlacement": false,

      "enableFeatureCorrelation": true,

      "reasoningFeatureList": "string",

      "enableU2I": true

    }

  }

}

```
This will return a build Id which we will refer to as buildId.

## 5. Deploy the container to App Services

We will now deploy the container to Linux App Services. Navigate to your App Service App, it will have the name recommapp[hash, select the *Docker Container* menu option. We now need to add the provisioned container registry and uploaded container details. Enter the following values:

* Image Source: 'Private Registry'
* Image and optional Tag: ```recommendationsacs[hash].azurecr.io/cadrecommendations:latest``` This is your registry
* Server URL: ```https://recommendationsacs[hash].azurecr.io```
* Username: The username that is in the Container Registry Access Keys blade
* Password: The password that is in the Container Registry Access Keys blade

This will now deploy your container to App Services, it may take a few minutes so browsing to the site will return an HTTP 503 status during deployment. It is recommended that you navigate to the *Overview* section and Stop and then Start your API App as opposed to clicking the Restart button - a preview gotcha.

Now if you navigate to the site and add */docs* on to the end of the URL, you will see the swagger definition for the API where you can discover the methods and test them in the test harness, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/swagger.png)
 
### Associate the Recommendations model URL with the API App

Our API app will check for two applications settings in order to be able to proxy the request to the Recommendations API, so we need to add these. Navigate to the *Application Settings* pane and add two entries in the 'App Settings' section, namely:

recommendationsAppURL  *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/recommend/item?itemIds=*
recommendationsKey [RecommendationsCS.key]

See below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/appsettings.png)

Save the settings and navigate to the *Overview* section and Stop and then Start your API App as opposed to clicking the Restart button - a preview gotcha.

# The Lab component

We want to front our Container API behind API management and use API Management to convert the JSON return from the Recommendations API to XML. We need to do this using the API Management JSON-->XML policy. We also want to redirect the resultant XML response from the Container API to the On-Premise Windows Server File system so that it can be written for the legacy web application. To do this we need create a Logic App that will connect to the On-Premise file system using the On-Premise Data Gateway.

To summarise, the steps include:
* Place the Container API behind API Management
* Convert the response JSON to XML using an API Management Policy
* Invoke a Logic App from an API Management Policy and pass the XML to it
* Connect to the On-Premise Data Gateway with a Logic App and commit the XML file to the file System. The file name must be the item id
* Only create the file if it does not exist, otherwise update the existing file
* Return a response to API Management so that it can be passed to the Container API

# The Lab solution

## SPOILER ALERT - this is the full solution, so only look here if you get stuck!

### Place the Container API behind API Management

We need to place the Container API behind API Management so that we can protect and manipulate the request without affecting the legacy File system. We also need to ensure that we can pass the item in that we want to find the next best item for to recommend to the user/customer. We need to pass this as a query parameter.

Navigate to API Management component provisioned within Azure, its name will be generated by default with the following format cadapim[hash].

Navigate to API Management component provisioned within Azure, its name will be generated by default with the following format cadapim[hash].

Click on the overview blade and select 'Publisher Portal' (note we will use the old portal while the new portal is still in preview), see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/apimpublisherportal.png)

This will navigate you to the Publisher portal, select Import API, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/importAPI.png)

Now enter the following values:

* Select "From Url"
* Paste your API URL from step 2 and add ``` /swagger ``` on the end e.g. http://recommapp[hash].azurewebsites.net/swagger
* Select "Swagger" as the specification format
* Select New API
* Type "RecommendationsAPI" into the Web API URL Suffix field
* Click Products and select "Unlimited"
* Click Save

See below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/recommendationsapi.png)

Now we need to change the mask of the URL so that we can pass the item id in as a query param as opposed to as part of the URI. Navigate to the *Operations* tab for your newly imported Recommendations API and select the Operation 'recommendations_getById'. 

In the signature section change the *URL Template* from:
```
/recommendations/{id}
```
to
```
/recommendations?id={id}
```
and Save, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/urltemplate.png)

You will now have imported an API that will now be accessible from the API Gateway. You can test it by clicking on Developer Portal, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/developerportal.png)

Click APIs --> Recommendations List API --> recommendations_getById --> Try It

This will take you to the test harness of API Management. Click the eyeball and copy the value in the field Ocp-Apim-Subscription-Key, this is your APIMKey which we will use extensively, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/apimkey.png)

You can test the API now and it should return values with a status of 200 Ok. Enter a value of 1 for the itemId, and you should see results similar to below if you used the coffee dataset:
```
Ocp-Apim-Trace-Location: https://apimgmtsttosp2ocbnmt9rah.blob.core.windows.net/apiinspectorcontainer/PGQhJTrzr8_m0zLxkafqSg2-1?sv=2015-07-08&sr=b&sig=EXe6rD%2BXYy0hZyJTfBzh0xLbBtX5Wg42o%2FcaD%2BXURPU%3D&se=2017-05-20T15%3A12%3A38Z&sp=r&traceId=3c06aa1150004ebcac21c676f888fdfe
Date: Fri, 19 May 2017 15:12:38 GMT
ETag: W/"18d-rykcPkfko79ztLN6R5uIow"
Set-Cookie: ARRAffinity=f6ac135c52b048d91b92cc79c27b985ddd7ffd33d5b3fa221fdb235825d7cb12;Path=/;Domain=recommapp83ikxnbrmeyocq.azurewebsites.net
Content-Length: 397
Content-Type: application/json; charset=utf-8

[
  {
    "recommendedItems": [
      {
        "items": [
          {
            "id": "3",
            "name": " Cappucino"
          }
        ],
        "rating": 0.502396748986747,
        "reasoning": [
          "Default recommendation for ' Americano'"
        ]
      },
      {
        "items": [
          {
            "id": "1",
            "name": " Latte"
          }
        ],
        "rating": 0.502396748986745,
        "reasoning": [
          "Default recommendation for ' Americano'"
        ]
      },
      {
        "items": [
          {
            "id": "4",
            "name": " Espresso"
          }
        ],
        "rating": 0.502396748986743,
        "reasoning": [
          "Default recommendation for ' Americano'"
        ]
      }
    ]
  }
]
```

### Convert the response JSON to XML using an API Management Policy

Let's add a policy to handle the conversion from JSON to XML. We need to add this in the API Response pipeline after the JSON has been retrieved from the backend event. 

In the Preview portal of the API Management component click the *APIS - PREVIEW* menu item and then 'Recommendations API' --> recommendations_getById --> Outbound Processing.

Here we will add the <json-to-xml> policy after the base section:

```
    <outbound>
        <base/>
        <json-to-xml apply="content-type-json" consider-accept-header="false"/>
    </outbound>
```
Click Save. Your pipeline should look like the image below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/jsonxml.png)

We can now test the API within the API Management portal, test the same method again as above but add the following header:

Accept : application/xml

You should now get a chunked xml response similar to below:
```
Transfer-Encoding: chunked
Ocp-Apim-Trace-Location: https://apimgmtsttosp2ocbnmt9rah.blob.core.windows.net/apiinspectorcontainer/PGQhJTrzr8_m0zLxkafqSg2-4?sv=2015-07-08&sr=b&sig=ezgXZOvffG4FLR3wtisqI5HUTu86PulE3BnVy%2BAsbEI%3D&se=2017-05-20T15%3A37%3A38Z&sp=r&traceId=2cd53d5ab5834c6483cea5f4ba1340b8
Date: Fri, 19 May 2017 15:37:38 GMT
ETag: W/"18d-rykcPkfko79ztLN6R5uIow"
Set-Cookie: ARRAffinity=f6ac135c52b048d91b92cc79c27b985ddd7ffd33d5b3fa221fdb235825d7cb12;Path=/;Domain=recommapp83ikxnbrmeyocq.azurewebsites.net
Content-Type: application/xml

<Document>
	<Array>
		<recommendedItems>
			<items>
				<id>3</id>
				<name> Cappucino</name>
			</items>
			<rating>0.502396748986747</rating>
			<reasoning>Default recommendation for ' Americano'</reasoning>
		</recommendedItems>
		<recommendedItems>
			<items>
				<id>1</id>
				<name> Latte</name>
			</items>
			<rating>0.502396748986745</rating>
			<reasoning>Default recommendation for ' Americano'</reasoning>
		</recommendedItems>
		<recommendedItems>
			<items>
				<id>4</id>
				<name> Espresso</name>
			</items>
			<rating>0.502396748986743</rating>
			<reasoning>Default recommendation for ' Americano'</reasoning>
		</recommendedItems>
	</Array>
</Document>

```
We have now successfully converted the outbound JSON with XML before it is sent back to the requestor.

### Invoke a Logic App from an API Management Policy and pass the XML to it

Now we want to invoke a Logic app inflight from API Management to synchronously write the RecommendationsAPI XML to the Legacy File system and return a response. 

Create a new Logic App within your Resource Group and in the same location as your other components.

The first thing we want to do is create a HTTP Request-Response Step, click save - you will receive an endpoint upon save - copy this value, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Personalisation/blob/master/images/logicreqresp.png)

Select 'Use this template'

![alt text](https://github.com/shanepeckham/CADScenario_Personalisation/blob/master/images/logicresprep2.png)

Now click Save and copy the endpoint

![alt text](https://github.com/shanepeckham/CADScenario_Personalisation/blob/master/images/copylogicurl.png)

Navigate back to your API Management component. Select *Named Values - PREVIEW* and click add, add the following value:
```
logicAppURL
```
Next add your Logic App endpoint that you copied into the 'Value' field and click 'This is a secret' tickbox and click 'Create', see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/apimnamedvalue.png)

We will now reference this 'Named Value' inflight within the API Management request.

In the Preview portal of the API Management component click the *APIS - PREVIEW* menu item and then 'Recommendations API' --> recommendations_getById --> Outbound Processing.

We retrieve the *Name Value* with the following syntax:
```
  var logicappurl = "{{LogicAppURL}}";
```
We can then set the url to the logicappurl variable using the <set-url> section.

Here we will add the <send-request> policy after the base section to invoke the Logic App url named value, see below:

```
 <send-request mode="new" response-variable-name="response" timeout="10" ignore-error="false">
            <set-url>
                        @{
                            var logicappurl = "{{LogicAppURL}}";
                            return logicappurl;
                        }
                    </set-url>
            <set-method>POST</set-method>
```

Click Save. Your pipeline should look like the image below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/sendrequest.png)

Let's test it now. When we run the API we can now see that the Logic app is onvoked by looking at the run history, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/logicrun.png)

Now we want to add a json schema to our request step so that we can work more easily with the logic app - we want to pass in the id of the item searched for and also pass the xml into the logic app so that we can write it to a file system. We can generate a json schema online - I have used https://jsonschema.net/#/editor which has generated the following schema I can use:
```
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "definitions": {},
  "id": "http://example.com/example.json",
  "properties": {
    "id": {
      "id": "/properties/id",
      "type": "integer"
    },
    "xml": {
      "id": "/properties/xml",
      "type": "string"
    }
  },
  "type": "object"
}
```
Add this to the request step. Now we need to pass in the XML response to the logic app as JSON. Yes, that is right, we are going to pass XML inside a JSON request.

Go back to the API Management <send-request> policy and add a JSON representation that conforms to the schema we have just added.

We want to add the following to the <send-request> policy. 

Firstly we will tell the Logic app to expect JSON, we will use the standard *Content-Type* header: 
```
 <set-header name="Content-Type" exists-action="override">
                <value>application/json</value>
            </set-header>
```

Now we will retrieve the xml within the response as a string and place it inside the JSON. 
```
var xml = context.Response.Body.As<string>(preserveContent: true); 
```
We will then create a JSON object and pass in the variables we want to pass. For now we will hard code the item id as we do not yet have this variable, see below.


```
 <set-header name="Content-Type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-body>
                 @{
                    var xml = context.Response.Body.As<string>(preserveContent: true); 
                    
                    var postBody = new JObject(
                            new JProperty("id", 1),
                            new JProperty("xml", xml)
                            ).ToString();
                            
                    return postBody;
                  }
            </set-body>

```

Your full outbound policy should now look like this:
```
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <base/>
    </backend>
    <outbound>
        <base/>
        <json-to-xml apply="content-type-json" consider-accept-header="false"/>
        <send-request mode="new" response-variable-name="response" timeout="10" ignore-error="false">
            <set-url>
                        @{
                            var logicappurl = "{{LogicAppURL}}";
                            return logicappurl;
                        }
                    </set-url>
            <set-method>POST</set-method>
             <set-header name="Content-Type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-body>
                 @{
                    var xml = context.Response.Body.As<string>(preserveContent: true); 
                    
                    var postBody = new JObject(
                            new JProperty("id", 1),
                            new JProperty("xml", xml)
                            ).ToString();
                            
                    return postBody;
                  }
            </set-body>
        </send-request>
    </outbound>
</policies>
```
We can now go and test it again, now in our logic app, if we open up the last run we should see the xml string in the Outputs step:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/xmloutput.png)

Now we can go and add a step to create the xml file on the Legacy File system. On the Logic app, after the Request step, add a *File System - Create File* step, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/logiccreatefile.png)

Select the root directory, name the file [id].xml and pass in the xml string, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/createfileparams.png)

You should see the following output:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/createfilesuccess.png)

And checking the Legacy OS file system we can see the file created:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/osfile.png)

#### Now let's get the itemId input parameter to pass it dynamically to the File System

We now want to capture the input parameter for id as it is passed in the initial request. We can declare a runtime variable to store this value in the inbound part of the request pipeline. 

The following is how we can declare the id variable and retrieve the *id* query parameter:
```
 <set-variable name="id" value="@(context.Request.Url.QueryString)"/>
```
Now let's replace the hardcoded id value in the outbound section with our new variable, we can retrieve a variable in-policy with the following syntax:
```
var id = context.Request.Url.Query.GetValueOrDefault("id");
```

We can now add this to the inbound section, your policy should now look like this:
```
<policies>
    <inbound>
        <base/>
        <set-variable name="id" value="@(context.Request.Url.QueryString)"/>
    </inbound>
    <backend>
        <base/>
    </backend>
    <outbound>
        <base/>
        <json-to-xml apply="content-type-json" consider-accept-header="false"/>
        <send-request mode="new" response-variable-name="response" timeout="10" ignore-error="false">
            <set-url>
                        @{
                            var logicappurl = "{{LogicAppURL}}";
                            return logicappurl;
                        }
                    </set-url>
            <set-method>POST</set-method>
            <set-header name="Content-Type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-body>
                 @{
                    var xml = context.Response.Body.As<string>(preserveContent: true); 
                    var id = context.Request.Url.Query.GetValueOrDefault("id");
                    
                    var postBody = new JObject(
                            new JProperty("id", id),
                            new JProperty("xml", xml)
                            ).ToString();
                            
                    return postBody;
                  }
            </set-body>
        </send-request>
    </outbound>
    <on-error>
        <base/>
    </on-error>
</policies>
```
With the following visual representation of the pipeline and policies:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/fullpipeline.png)

Now we can change the value in the logic app to accept the id field dynamically. If all works we should now get an error from the logic app as if we invoke the API with id=1 we will get the following error:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/fileexists.png)

We now need to add exception handling to our logic app to ensure that we update a file if it already exists. Now add a  *File System - Update File* step after the *File System - Create File*.

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/updatefile.png)

We want to determine the file name dynamically so that we can update it, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/triggerbody.png)

This is what my filename contents look like:

```
\\legacyxml\SHOPPINGXML\@{triggerBody()?['id']}.xml
```

Now we want to make sure that the Update Step only runs if the Create File step fails. We can do by adding exception handling to the logic app. We can amend the *Run After* step of the Update step to only run if the Create Step fails, see below:

```
"runAfter": {
	    "Create_file": [
		"Failed"
	    ]
},
```
So my logic app Update Step in the Code View now looks like this:
```
  "Update_file": {
                "inputs": {
                    "body": "@triggerBody()?['xml']",
                    "host": {
                        "api": {
                            "runtimeUrl": "https://logic-apis-westeurope.azure-apim.net/apim/filesystem"
                        },
                        "connection": {
                            "name": "@parameters('$connections')['filesystem']['connectionId']"
                        }
                    },
                    "method": "put",
                    "path": "/datasets/default/files/@{encodeURIComponent(encodeURIComponent('XFxsZWdhY3l4bWxcU0hPUFBJTkdYTUxcMS54bWw='))}"
                },
                "metadata": {
                    "XFxsZWdhY3l4bWxcU0hPUFBJTkdYTUxcMS54bWw=": "\\\\legacyxml\\SHOPPINGXML\\@{triggerBody()?['id']}.xml"
                },
                "runAfter": {
                    "Create_file": [
                        "Failed"
                    ]
                },
                "type": "ApiConnection"
            }
```

Now if you run the Logic app with id = 1 you should get the following run details:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/exception.png)

You can now test the process with id = 2 and you should see the following:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/createnew.png)

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/newfile.png)









