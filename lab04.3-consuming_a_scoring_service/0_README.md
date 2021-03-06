# Consuming a scoring service

This hands-on lab guides you through consuming a Machine Learning scoring service using [Azure Machine Learning Services](https://docs.microsoft.com/en-us/azure/machine-learning/preview/overview-what-is-azure-ml) with the Azure Machine Learning Workbench. 

In this workshop, you will:
- Understand how to consume a deployed model from a Web API

***NOTE:*** There are several pre-requisites for this course, including an understanding and implementation of: 
  *  Programming using an Agile methodology
  *  Machine Learning and Data Science
  *  Intermediate to Advancced Python programming
  *  Familiarity with Web Services and API Programming
  *  Familiarity with [Swagger](https://github.com/swagger-api/swagger-codegen)

There is a comprehensive Learning Path you can use to prepare for this course [located here](https://github.com/Azure/learnAnalytics-CreatingSolutionswiththeTeamDataScienceProcess-/blob/master/Instructions/Learning%20Path%20-%20Creating%20Solutions%20with%20the%20Team%20Data%20Science%20Process.md).

### Lab 1: Creating a realtime service

***You do not have to do Lab 1 if you already have a cluster based service.***

In this lab, you'll create an experiment based on Churn Prediction and create a realtime cluster based service. The reason we use cluster is because key operations are not supported for local services.

You should now be familiar with many of the below steps from the previous labs:

**Generate the scoring and schema files**

Run ```python churn_schema_gen.py``` from CLI to create service_schema.json.

**Create the environment in cluster mode**

```az ml env setup -n <new deployment environment name> --location <e.g. eastus2> -c```

**Create a Model Management account**

```az ml account modelmanagement create --location <e.g. eastus2> -n <new model management account name> -g <existing resource group name> --sku-name S1```

**Set the Model Management account**

```az ml account modelmanagement set -n <youracctname> -g <yourresourcegroupname>```

**Set the environment**

```az ml env set -n <deployment environment name> -g <existing resource group name>```

**Create the real-time web-service**

```az ml service create realtime -f score.py --model-file model.pkl -s service_schema.json -n <name> -r python```

### Lab 2: Consume the web-service

In this lab, we will send a request to the real-time web service created by following the below steps:

**Get the service key**

```az ml service keys realtime -i <web service id>```

You will get PrimaryKey and SecondaryKey with this command. You can use either of the keys for authorization when consuming the web-service.

**Get the service URL**

Obtain the service URL from the below command

```az ml service usage realtime -i <service name>```

The URL can be obtained from the sample CURL call. For example, the url is http://40.70.13.110:80/api/v1/service/churncluster333/score in the below sample CURL call:

```curl -X POST -H "Content-Type:application/json" -H "Authorization:Bearer <key>" --data "{\"input_df\": [{\"callfailurerate\": 0, \"education\": \"Bachelor or equivalent\", \"usesinternetservice\": \"No\", \"gender\": \"Male\", \"unpaidbalance\": 19, \"occupation\": \"Technology Related Job\", \"year\": 2015, \"numberofcomplaints\": 0, \"avgcallduration\": 663, \"usesvoiceservice\": \"No\", \"annualincome\": 168147, \"totalminsusedinlastmonth\": 15, \"homeowner\": \"Yes\", \"age\": 12, \"maritalstatus\": \"Single\", \"month\": 1, \"calldroprate\": 0.06, \"percentagecalloutsidenetwork\": 0.82, \"penaltytoswitch\": 371, \"monthlybilledamount\": 71, \"churn\": 0, \"numdayscontractequipmentplanexpiring\": 96, \"totalcallduration\": 5971, \"callingnum\": 4251078442, \"state\": \"WA\", \"customerid\": 1, \"customersuspended\": \"Yes\", \"numberofmonthunpaid\": 7, \"noadditionallines\": \"\\N\"}]}" http://40.70.13.110:80/api/v1/service/churncluster333/score```

**Swagger document**

If the service API schema was supplied, the service endpoint would expose a Swagger document at http://<ip>/api/v1/service/<service name>/swagger.json. The Swagger document can be used to automatically generate the service client and explore the expected input data and other details about the service.

**Call the web service using curl**

Obtain the curl command from ```az ml service usage realtime -i <service name>``` and replace &lt;key&gt; with PrimaryKey / SecondaryKey.

**Call the web service using Python**

Use Python to send a request to your real-time web service. 

1. Copy the following code sample to a new Python file.
2. Update the data, url, and api_key parameters. For local web services, remove the 'Authorization' header.
3. Run the code. 

```
import requests
import json

data = "{\"input_df\": [{\"feature1\": value1, \"feature2\": value2}]}"
body = str.encode(json.dumps(data))

url = 'http://<service ip address>:80/api/v1/service/<service name>/score'
api_key = 'your service key' 
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}

resp = requests.post(url, data, headers=headers)
resp.text
```

***NOTE:*** Some of the field values to test can potentially have backslashes in input_df when copying from ```az ml service usage realtime -i <service name>``` execution results. For example, \\"noadditionallines\\": \\"\\\N\\". In the python code, this would need to be changed to \\"noadditionallines\\": \\"\\\\\\\\N\\".

### (Optional) Lab 3: Consume the web-service using C#

Use the service URL to send a request from a C# Console App. 

In Visual Studio, create a new Console App: 
- In the menu, click, File -> New -> Project
- Under Visual Studio C#, click Windows Class Desktop, then select Console App.
- Enter MyFirstService as the Name of the project, then click OK.
- In Project References, set references to System.Net, and System.Net.Http.
Click Tools -> NuGet Package Manager -> Package Manager Console, then install the Microsoft.AspNet.WebApi.Client package.
- Open Program.cs file, and replace the code with the following code:
Update the SERVICE_URL and API_KEY parameters with the information from your web service.
- Run the project.

````C#
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;
using Newtonsoft.Json;

namespace MyFirstService
{
    class Program
    {
        // Web Service URL (update it with your service url)
        private const string SERVICE_URL = "http://<service ip address>:80/api/v1/service/<service name>/score";
        private const string API_KEY = "your service key";

        static void Main(string[] args)
        {
            Program.PostRequest();
            Console.ReadLine();
        }

        private static void PostRequest()
        {
            HttpClient client = new HttpClient();
            client.BaseAddress = new Uri(SERVICE_URL);
            //For local web service, comment out this line.
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", API_KEY);

            var inputJson = new List<RequestPayload>();
            RequestPayload payload = new RequestPayload();
            List<InputDf> inputDf = new List<InputDf>();
            inputDf.Add(new InputDf()
            {
                feature1 = value1,
                feature2 = value2,
            });
            payload.Input_df_list = inputDf;
            inputJson.Add(payload);

            try
            {
                var request = new HttpRequestMessage(HttpMethod.Post, string.Empty);
                request.Content = new StringContent(JsonConvert.SerializeObject(payload));
                request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                var response = client.SendAsync(request).Result;

                Console.Write(response.Content.ReadAsStringAsync().Result);
            }
            catch (Exception e)
            {
                Console.Out.WriteLine(e.Message);
            }
        }
        public class InputDf
        {
            public double feature1 { get; set; }
            public double feature2 { get; set; }
        }
        public class RequestPayload
        {
            [JsonProperty("input_df")]
            public List<InputDf> Input_df_list { get; set; }
        }
    }
}
````

## Workshop Completion

In this workshop you learned how to:
- Understand how to consume a deployed model from a Web API

You may now decommission and delete the following resources if you wish:
  * The Azure Machine Learning Services accounts and workspaces, and any Web API's
  * Any Data Science Virtual Machines you have created. NOTE: Even if "Shutdown" in the Operating System, unless these Virtual Machines are "Stopped" using the Azure Portal you are incurring run-time charges. If you Stop them in the Azure Portal, you will be charged for the storage the Virtual Machines are consuming.