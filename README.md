# 504_serverless_functions

## Overview
This project deploys a HTTP function across Google Cloud Platform(GCP) and Microsoft Azure that classifies HbA1c levels as normal or abnormal based on published clinical reference ranges. 

## Lab Rules
Lab Chosen: HbA1c
* Rule Implemented

  * Plain English: An HbA1c value less than 5.7 is considered normal. An HbA1c value of 5.7 or higher is considered abnormal indicating prediabetes or diabetes.
  * Formula/Threshold
    * Normal: HbA1c < 5.7
    * Abnormal: HbA1c > 5.7 

* Citation
    * Eyth, E., Zubair, M., & Naik, R. (2025). Hemoglobin A1C. *StatPearls Publishing LLC*. Retrieved from https://www.ncbi.nlm.nih.gov/books/NBK549816/ 

## Cloud Environments and Regions Used
- Google Cloud Platform(GCP): Europe West 1
- Microsoft Azure: Canada Central 1

## Endpoint URLs
- GCP Endpoint: https://hha504-serverlesshw-788221590655.europe-west1.run.app 
    - Method: POST

- Microsoft Azure Endpoint: https://python-serverless-gcffbjdngbgcbmg4.canadacentral-01.azurewebsites.net/api/http_trigger1?code=p3OHrvREOjIUXvJaHD9nF0sNkxyZPqBsrsDy4F5UXi6wAzFu1dZC8A== 
    - Method: POST

## Deployment Steps Executed

### GCP

1. In GCP, search Cloud Functions
2. Click Create Function
3. Use whatever region
4. Change Runtime type(language to use) to Python 3.13 or whatever version you choose. 
5. Under Authentication, allow public access
6. Once created, open the URL in a different browser and this should be seen before updating the code:


7. Update requirements.tx
```
functions-framework==3.*
```
8. Update the main.py tab
    - Replace content with the following code:
``` 
import functions_framework
import json 

@functions_framework.http
def hba1c_classifier(request):
    try:
        # Get JSON input
        request_json = request.get_json(silent=True)
        if not request_json or 'hba1c' not in request_json:
            return json.dumps({"error": "Missing hba1c value"}), 400

        hba1c = float(request_json['hba1c'])
        # Classification logic
        result = "normal" if hba1c < 5.7 else "abnormal"
        return json.dumps({"hba1c": hba1c, "result": result}), 200
    except (ValueError, TypeError):
        return json.dumps({"error": "Invalid hba1c value"}), 400
```
9. Click the Deploy button. You will see a green checkmark with "Completed" when deployment is successful. 
10. Copy the Trigger URL which will be used for testing. 


### Microsoft Azure

1. In the search bar in the Azure Portal, search for Function App and click create
2. Configure the Function App;
    - Pick a resource group
    - Name the Function App. In this case the name given was "python-serverless"
    - Choose runtime stack preferabvly Python 3.13
    - Under Hosting options, click Consumption
3. Click Create.
4. Create the HTTP trigger function
5. Click on the function_app.py tab
6. Replace all the content with the following:
```
import azure.functions as func
import logging
import json

app = func.FunctionApp(http_auth_level=func.AuthLevel.FUNCTION)

@app.route(route="http_trigger1")
def http_trigger1(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    try:
        req_body = req.get_json()
        if 'hba1c' not in req_body:
            return func.HttpResponse(
                json.dumps({"error": "Missing hba1c value"}),
                status_code=400,
                mimetype="application/json"
            )
        
        hba1c = float(req_body['hba1c'])
        result = "normal" if hba1c < 5.7 else "abnormal"

        return func.HttpResponse(
            json.dumps({"hba1c": hba1c, "result": result}),
            status_code=200,
            mimetype="application/json"
        )

    except (ValueError, TypeError):
        return func.HttpResponse(
            json.dumps({"error": "Invalid hba1c value"}),
            status_code=400,
            mimetype="application/json"
        )
``` 
7. To save the file, click Save at the top left corner. 

#### Test the Function in the Azure Portal

1. In the function editor, click the Test/Run button
2. In the "Body " field enter
```
json
{ 
    "hba1c: 4
}
```
3. Click Run
4. The expected output is 
```
{"hba1c": 4.0, "result": "normal"}
```
## Example Requests 

## Cloud Comparison 
GCP felt more straightforward to me for initial deployment. I took less time to delpoy the commands for GCP.  The monitoring logs are in the Cloud logging interface with good visibility. But it does requrie a lot more setup compared to Azure. 
Azure required more code which is why it took a little more time to complete. The difference that I like with Azure is that you could click the test/run option and immediately deploy, and see what's wrong with your code. Everything requires less clicks compared to GCP. But for this project, I still prefer GCP because of how quick the deployments are but for troubleshooting, Azure is better.  

## Recording
