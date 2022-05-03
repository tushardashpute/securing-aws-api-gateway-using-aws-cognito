# securing-aws-api-gateway-using-aws-cognito
securing-aws-api-gateway-using-aws-cognito

Steps:
 1. Create a AWS Cognito user pool and configure OAuth agents
 2. Create Lambda Functions (getLambda/PutLambda/DeleteLambda)
 3. Create API gateway, Create methods(GET/POST/DELETE) and integrate it with Lambda Funtion which we created
 4. Test the Gateway endpoint 
 5. Create authorization as a cognito in API Gateway
 6. Add authorization to each method which we created 


**1.Create a AWS Cognito user pool and configure OAuth agents**

- Login to AWS Management console and navigate to Cognito service
- Select “Manage your user pools” and click “Create a user pool”
- Enter a pool name and select “Review defaults”. Then select “Create pool”.

<img width="1500" alt="image" src="https://user-images.githubusercontent.com/74225291/166409845-e416cb5d-8826-49c2-a9ac-d6d0c59d940a.png">

- Navigate to “General Settings > App clients” and select “Add an app client”
- Enter a “App client name” and select “Generate client secret” checkbox. Then “Create app client”. Note down the “App client id” and “App client secret” values displayed in next page.

<img width="1509" alt="image" src="https://user-images.githubusercontent.com/74225291/166409950-5ef74568-af6e-43df-9621-97e55bfe1093.png">

- Go to “Domain name” and enter your own domain name. It can be any name like test, test123 etc. You can check if the domain is available or not. Let us assume that domain name is api-product. So, the URL would be https://testLambdaApi.auth.us-east-1.amazoncognito.com

<img width="1506" alt="image" src="https://user-images.githubusercontent.com/74225291/166410154-101179e0-5be4-47ce-9bb4-975d832c252a.png">

Go to “Resource Servers” and “Add a resource server”
Enter “Name” and “Identifier”. It can be any value. Also, add 3 scopes and save changes.

- read_lambda
- create_lambda
- delete_lambda

<img width="1476" alt="image" src="https://user-images.githubusercontent.com/74225291/166410421-3fa3a421-2b8f-4076-9c3e-f345f5b37e41.png">

- Go to “App client settings” and you should see the configuration page for new App client. For “Enabled Identity providers” , select “Cognito User pool” checkbox. Then select “Client credentials” checkbox for “Allowed OAuth flows”. Select all the scopes for “Allowed custom scopes” and save changes. If certain clients should have only “read_lambda” scope, then select only that checkbox.

<img width="1504" alt="image" src="https://user-images.githubusercontent.com/74225291/166410549-0a449ad6-151c-45b1-8cec-3a12c148db17.png">

Now, we have successfully setup a OAuth2 agent in Cognito. Below CURL command should return an access token


  //Replace app client id and secret accordingly. Also, the URL will change if you had selected a different domain name
  curl -X POST --user <app client id>:<app client secret> 'https://testlambdaapi.auth.us-east-1.amazoncognito.com/oauth2/token?grant_type=client_credentials' -H 'Content-Type: application/x-www-form-urlencoded'

  Tdaspute@INPNL000868 Downloads % curl -X POST --user 4jh9topp45s1fc8n4g64shtls2:1jq0ms2h578rrc5v01r41362dec503cpvro886oi1ohgopq7f9iu 'https://testlambdaapi.auth.us-east-1.amazoncognito.com/oauth2/token?grant_type=client_credentials' -H 'Content-Type: application/x-www-form-urlencoded'

  {"access_token":"eyJraWQiOiI3ak5xZ0hoZ3JCK0F3RGZcLzZ0bEdhZEJUTUhvbEJHdktNdXBqaTBBY2dRZz0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI0amg5dG9wcDQ1czFmYzhuNGc2NHNodGxzMiIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoibGFtYmRhQXBpXC9jcmVhdGVfbGFtYmRhIGxhbWJkYUFwaVwvcmVhZF9sYW1iZGEgbGFtYmRhQXBpXC9kZWxldGVfbGFtYmRhIiwiYXV0aF90aW1lIjoxNjUxNTU3ODg4LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9CUmFoeFBmSE0iLCJleHAiOjE2NTE1NjE0ODgsImlhdCI6MTY1MTU1Nzg4OCwidmVyc2lvbiI6MiwianRpIjoiYzZhMGVhYzktMTBjZC00NmYzLWIzMGQtNDRmMmE2YWZiMDg4IiwiY2xpZW50X2lkIjoiNGpoOXRvcHA0NXMxZmM4bjRnNjRzaHRsczIifQ.A2ryAxpC-y3ltfESgU-Cb6qXRTr3Ptt0hmMPycy5j-OUc3beeA9umhzLbuVwnXkH-upkbmLwvjm53-d8yJQTVjI93hgMJHfeFdwM2orXGsc-R7Q4NEekySmyZDhttIVtjh3Bx6e_Q6LmiWtGf6m6jZ8O0G1nvSy_w0ZCI4R5FCcCKUg0-bIUqM-Dr-l4dOhKptt5di-upOvakzx429z-w6qYSMl5mH1uYW0JubPcfm8HzlnRo-dOsAgbq84-hx9gtgrEErfkzeAG0JmWSy9XtOH5qJ9w9vLEhltrkLyChdaeq2OfhdijtBQZeGhUO01Rd492CoEb7UxNOJ60Ae2yqg","expires_in":3600,"token_type":"Bearer"}
