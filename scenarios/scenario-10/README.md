## Scenario 10 - Apply interceptors to an API

- This scenario describes how to apply interceptors to carry out transformations and mediations on the requests and responses.
- First, we need to implement custom request interceptors and response interceptors. We have provided sample request and response interceptors in scenario-10. If you want to learn more about implementing custom interceptors you can refer the document [adding interceptors.](https://docs.wso2.com/display/MG300/Adding+Interceptors)
- Then we need to Initialize a new API project and add the custom interceptor files in interceptors folder.
- We need to refer the interceptors in swagger definition in order to apply them on the requests and responses.
- Finally, we will invoke the API and observe how the added interceptors act on requests and responses.

 ***Important:***
> Follow the main README and deploy the apim-operator and configuration files. Make sure to set the analyticsEnabled to "true" and deploy analytics secret with credentials to analytics server and certificate, if you want to check analytics.


 ##### Deploying the artifacts
 
 - Init the API project using CLI. This will Initialize a new API project in same directory.
 
     ```
     >> apictl init petstore-int --oas=interceptor_swagger.yaml
    
    Output:    
    Initializing a new WSO2 API Manager project in ./product-apim-tooling/import-export-cli/build/target/apimcli/petstore-int
    Project initialized
    Open README file to learn more
     ```
  
 - Copy the _modifyResponse.bal_ and _validateQueryPrams.bal_ files in scenario-10 into the interceptors folder in petstore-int/Interceptors path.
 
 - We have defined a function validateResponse in _modifyResponse.bal_ file which check whether the response payload is json and if so then the payload will be modified as follows. 
 
    ```
    "pets": payload,
    "length": payload length
    ```

 - In _validateQueryPrams.bal_ file we have defined a function validateRequest which check the query parameter "status" of the request URL and validate the request accordingly. 
 
 - In order to refer interceptor functions in swagger definition we have already modified the swagger definition by adding following OpenAPI extensions with interceptor function names.
     - _validateQueryPrams.bal_ file consists with _validateRequest_ function which modify the request flow. we have added the function name _validateRequest_ in the swagger definition as follow.
     
        ```
        x-wso2-request-interceptor: validateRequest
        ```
     - _modifyResponse.bal_ file consists with _validateResponse_ function which modify the response flow. we have added the function name _validateResponse_ in the swagger definition as follow.
     
        ```
        x-wso2-response-interceptor: validateResponse
        ```
     
- Execute the following to expose pet-store as an API.

- Create the API

    ```
    >> apictl add api -n petstore-int --from-file=petstore-int
    
    Output:
    creating configmap with swagger definition
    configmap/petstore-int-swagger created
    creating configmap with interceptors
    configmap/petstore-int-interceptors created
    api.wso2.com/petstore-int created
    ```
  

- Get service details to invoke the API. (Please wait until the external-IP is populated in the corresponding service)

    ```
    >> apictl get services
    
    Output:
    NAME            TYPE           CLUSTER-IP   EXTERNAL-IP       PORT(S)                         AGE
    petstore-int   LoadBalancer   10.83.4.44   104.197.114.248   9095:30680/TCP,9090:30540/TCP   8m20s
    ```
    
    - You can see petstore-int service has been exposed as a managed API.
    - Get the external IP of the managed API's service
 
- Invoking the API

    ```
   TOKEN=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlpqUm1ZVE13TlRKak9XVTVNbUl6TWpnek5ESTNZMkl5TW1JeVkyRXpNamRoWmpWaU1qYzBaZz09In0.eyJhdWQiOiJodHRwOlwvXC9vcmcud3NvMi5hcGltZ3RcL2dhdGV3YXkiLCJzdWIiOiJhZG1pbkBjYXJib24uc3VwZXIiLCJhcHBsaWNhdGlvbiI6eyJvd25lciI6ImFkbWluIiwidGllciI6IlVubGltaXRlZCIsIm5hbWUiOiJzYW1wbGUtY3JkLWFwcGxpY2F0aW9uIiwiaWQiOjMsInV1aWQiOm51bGx9LCJzY29wZSI6ImFtX2FwcGxpY2F0aW9uX3Njb3BlIGRlZmF1bHQiLCJpc3MiOiJodHRwczpcL1wvd3NvMmFwaW06MzIwMDFcL29hdXRoMlwvdG9rZW4iLCJ0aWVySW5mbyI6e30sImtleXR5cGUiOiJQUk9EVUNUSU9OIiwic3Vic2NyaWJlZEFQSXMiOltdLCJjb25zdW1lcktleSI6IjNGSWlUM1R3MWZvTGFqUTVsZjVVdHVTTWpsUWEiLCJleHAiOjM3MTk3Mzk4MjYsImlhdCI6MTU3MjI1NjE3OSwianRpIjoiZDI3N2VhZmUtNTZlOS00MTU2LTk3NzUtNDQwNzA3YzFlZWFhIn0.W0N9wmCuW3dxz5nTHAhKQ-CyjysR-fZSEvoS26N9XQ9IOIlacB4R5x9NgXNLLE-EjzR5Si8ou83mbt0NuTwoOdOQVkGqrkdenO11qscpBGCZ-Br4Gnawsn3Yw4a7FHNrfzYnS7BZ_zWHPCLO_JqPNRizkWGIkCxvAg8foP7L1T4AGQofGLodBMtA9-ckuRHjx3T_sFOVGAHXcMVwpdqS_90DeAoT4jLQ3darDqSoE773mAyDIRz6CAvNzzsWQug-i5lH5xVty2kmZKPobSIziAYes-LPuR-sp61EIjwiKxnUlSsxtDCttKYHGZcvKF12y7VF4AqlTYmtwYSGLkXXXw
    ```
   
    ```
     >> curl -X GET "https://<External_IP>:9095/petstoreint/v1/pet/findByStatus?status=available"  -H "accept: application/xml" -H "Authorization:Bearer $TOKEN" -k
    ```
    
    - Once you execute the above command, it will call to the managed API (petstore-int), which then call its endpoint(https://petstore.swagger.io/v2). If the request is success, you would be able to see the response as below.
    
        ```
        <?xml version="1.0" encoding="UTF-8" standalone="yes"?><pets><Pet><category><id>12531424102019</id><name>dragon name string</name></category><id>32132142</id><name>dragon</name><photoUrls><photoUrl>string</photoUrl></photoUrls><status>available</status><tags><tag><id>12531424102019</id><name>string</name></tag></tags></Pet><Pet><category><id>0</id><name>dldlld</name></category><id>32132143</id><name>doggie</name><photoUrls><photoUrl>string</photoUrl></photoUrls><status>available</status><tags><tag><id>0</id><name>string</name></tag></tags></Pet> ……
        ```
        
    - Then invoke the API without "status" query parameter to test the request interceptor as follows.
    
        ```
        >> curl -X GET "https://<External_IP>:9095/petstoreint/v1/pet/findByStatus"  -H "accept: application/xml" -H "Authorization:Bearer $TOKEN" -k
        ```
        
    - Once you execute the above command, you will get the error message by indicating invalid request as follows.
    
        ```
        {"error":"Missing a required parameter"}
        ```
    - Then invoke the API with "accept: application/json" header to test the response interceptor as follows.
    
        ```
        >> curl -X GET "https://<External_IP>:9095/petstoreint/v1/pet/findByStatus?status=available"  -H "accept: application/json" -H "Authorization:Bearer $TOKEN" -k -v
        ```
        
    - Once you execute the above command, you will get the response as follows.
    
        ```
        {
        "pets":[{"id":32132142, "category":{"id":12531424102019, "name":"dragon name string"}, "name":"dragon", "photoUrls":["string"], "tags":[{"id":12531424102019, "name":"string"}], "status":"available"}, …. ,{“id":1, "category":{"id":0, "name":"hhhhhhh"}, "name":"mm", "photoUrls":["string"], "tags":[{"id":0, "name":"string"}], "status":"available"}], 
        "length":419
        }
        ```