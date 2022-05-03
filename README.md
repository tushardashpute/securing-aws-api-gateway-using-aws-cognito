# securing-aws-api-gateway-using-aws-cognito
securing-aws-api-gateway-using-aws-cognito

Steps:
 1. Create a AWS Cognito user pool and configure OAuth agents
 2. Create Lambda Functions (getLambda/PutLambda/DeleteLambda)
 3. Create API gateway, Create methods(GET/POST/DELETE) and integrate it with Lambda Funtion which we created
 4. Test the Gateway endpoint 
 5. Create authorization as a cognito in API Gateway
 6. Add authorization to each method which we created 
 7. Testing

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

 
 You can copy and paste the access_token value in https://jwt.io/ website. You should see all the 3 scopes in the token by default — “scope”: “lambdaApi/create_lambda lambdaApi/read_lambda lambdaApi/delete_lambda”

 Cognito follows the OAuth2 specification. You can try passing a specific scope in the CURL command and check the token.
 
//access_token returned with below CURL command should return only lambdaApi/create_lambda scope

 curl -X POST --user <app client id>:<app client secret> 'https://testlambdaapi.auth.us-east-1.amazoncognito.com/oauth2/token?grant_type=client_credentials&scope=product-api/delete_product' -H 'Content-Type: application/x-www-form-urlencoded'
 
 
 **2.Create Lambda Functions (getLambda/PutLambda/DeleteLambda)**

- Login to AWS Management console and navigate to AWS Lambda service.
- Create a new function. In this screen, select author from scratch .Give name as getLambda, Select runtime as python 3.9.
 
 <img width="1440" alt="image" src="https://user-images.githubusercontent.com/74225291/166412778-4be48a7a-41eb-46b1-9a11-36d33b6b3c67.png">

 Now goto code and write below code.
 
   import json

  def lambda_handler(event, context):
      # TODO implement
      return {
          'statusCode': 200,
          'body': json.dumps('Hello from Get Lambda!')
      }

 Deploy it and test it.
 
 <img width="1402" alt="image" src="https://user-images.githubusercontent.com/74225291/166413018-971a939b-78b3-4ed1-8478-212a1ddb1dfb.png">

 <img width="1444" alt="image" src="https://user-images.githubusercontent.com/74225291/166413044-e6aee083-36d6-4f25-b90c-2a8ab72f4869.png">

 
Similarly create 2 more lambdaFuntions, putLambda and DeleteLambda.
 
 <img width="1444" alt="image" src="https://user-images.githubusercontent.com/74225291/166413115-a3de2a11-f72a-4387-858d-dc3e55033a6e.png">

**3. Create API gateway, Create methods(GET/POST/DELETE) and integrate it with Lambda Funtion which we created**
 
- Login to AWS Management console and navigate to API Gateway service.
- Create API. In this screen, select REST API --> Build Give name as LAmbDA-API.

 <img width="1311" alt="image" src="https://user-images.githubusercontent.com/74225291/166413290-7efba689-2e77-4ac3-95a6-1dc35943df9d.png">

 <img width="1523" alt="image" src="https://user-images.githubusercontent.com/74225291/166413715-cfb7824a-8cbd-4e8d-affc-e4c8105dcbef.png">
 
 Goto Resources --> Action -->  add resource
 
 <img width="1525" alt="image" src="https://user-images.githubusercontent.com/74225291/166413849-99c3e667-8a72-4be5-ae64-4d985c7fcce3.png">
 
 Goto resource --> Action --> Add method (GET/POST/DELETE)
 
 <img width="1507" alt="image" src="https://user-images.githubusercontent.com/74225291/166414428-fd096a71-ba1d-4a65-8151-db41def0fb3e.png">
 
 Now integrate the Lambda with it.
 
 <img width="1520" alt="image" src="https://user-images.githubusercontent.com/74225291/166414565-a7c30c1c-56d6-413d-aa62-f75daf3882e2.png">

 Do same for remaining two methods as well.
 
 Now deploy the API.
 
 <img width="1472" alt="image" src="https://user-images.githubusercontent.com/74225291/166414690-b54353a6-2e86-44e9-ae77-6904de9c98c2.png">

 
 <img width="723" alt="image" src="https://user-images.githubusercontent.com/74225291/166414723-a7667b18-a987-48fe-80b4-6ea54cd56ca1.png">


 
**4. Test the Gateway endpoint** 

 Now access the API using the API URL which we get after deployment.
 
 Gateway_api_url/resource_name
 
 <img width="910" alt="image" src="https://user-images.githubusercontent.com/74225291/166418763-2ff2e186-568d-41f4-bc41-1477e4992287.png">

 <img width="910" alt="image" src="https://user-images.githubusercontent.com/74225291/166415306-c9967f8e-a960-4d6c-9b6a-244642f6273c.png">

<img width="910" alt="image" src="https://user-images.githubusercontent.com/74225291/166418836-e1844b1a-4532-4957-8088-7748bc8e27fd.png">

 
**5. Create authorization as a cognito in API Gateway**
 
 Goto API gateway --> Authorizores and create new cognito authorizer
 
 <img width="910" alt="image" src="https://user-images.githubusercontent.com/74225291/166418947-4d529a08-aaa3-4eec-bb9c-b18b5af2abb4.png">

**6. Add cognito authorization to each method which we created**  

 - Go to “Resources” and select “GET” method. Select “Method Request” configuration on right pane.
 - Select “Cognito_Authorizer” in “Authorization” drop-down. That should automatically add a new field “OAuth Scopes”. Enter “lambdaApi/create_lambda” scope and save using the tick mark.
 
 <img width="1531" alt="image" src="https://user-images.githubusercontent.com/74225291/166421620-5e831c98-bb60-4cd2-a44a-239c0f66ecd4.png">
 
 - Similarly, map DELETE method to “lambdaApi/delete_lambda” and POST method to “lambdaApi/create_lambda”
 
 <img width="1036" alt="image" src="https://user-images.githubusercontent.com/74225291/166421852-f62711b7-e039-4468-9c1e-5a26dadd3e34.png">

 <img width="1067" alt="image" src="https://user-images.githubusercontent.com/74225291/166421938-eb0f9971-4df6-428d-afab-7b7b5963c1b4.png">
 
-  Select “Actions > Deploy API” and select “Deployment stage” as “dev”. This should deploy the latest changes in these APIs.

**7. Testing**
 
 Now, let us test this API using the access_token obtained from Cognito
 
  curl -X GET 'https://h7l9qseapd.execute-api.us-east-1.amazonaws.com/Dev/lambdaapi' -H 'Content-Type: application/json'
    {"message":"Unauthorized"}%      

Now we will use the access token, to access the API endpoint.
 
   % curl -X POST --user 4jh9topp45s1fc8n4g64shtls2:1jq0ms2h578rrc5v01r41362dec503cpvro886oi1ohgopq7f9iu 'https://testlambdaapi.auth.us-east-1.amazoncognito.com/oauth2/token?grant_type=client_credentials' -H 'Content-Type: application/x-www-form-urlencoded'

     {"access_token":"eyJraWQiOiI3ak5xZ0hoZ3JCK0F3RGZcLzZ0bEdhZEJUTUhvbEJHdktNdXBqaTBBY2dRZz0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI0amg5dG9wcDQ1czFmYzhuNGc2NHNodGxzMiIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoibGFtYmRhQXBpXC9jcmVhdGVfbGFtYmRhIGxhbWJkYUFwaVwvcmVhZF9sYW1iZGEgbGFtYmRhQXBpXC9kZWxldGVfbGFtYmRhIiwiYXV0aF90aW1lIjoxNjUxNTY1NDk3LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9CUmFoeFBmSE0iLCJleHAiOjE2NTE1NjkwOTcsImlhdCI6MTY1MTU2NTQ5NywidmVyc2lvbiI6MiwianRpIjoiNTNhMGRhNjgtZWRkZC00MzBiLWFkNWItNTBkYmU5ZjAyM2QyIiwiY2xpZW50X2lkIjoiNGpoOXRvcHA0NXMxZmM4bjRnNjRzaHRsczIifQ.AtLo5xlxNx8C0vquzUMDPfGvivXIhfKOM3QFEf8PWb-phX2q86mxQMMHyYQ95gtEdb0Qdx7FrEc5zRdg94urvLyDprvlJospEa8x7UvGWO_9iJ-hp54QQFW4ySLjX3DMUQPUD7HXKeR9ycQmOyrBAA_EGHs1hEq2Vwc51X3FNTPbacMjyWMq7pMxtzwCKkt2Q9i6oOx85d7qxxJsJ_cAoYm1U_bGXO1jCYXUCBQsK7DcnDAKp4PSYMvvh2D4Q4vXx1WGOpLeb-RQyqeQNqMXUotbkxJV_TwMsnkCucHMukEaLBXKKaLiOdMawXd0WxbTV9P-dJOVUhZxzBvAnMWhGw","expires_in":3600,"token_type":"Bearer"}
 
 
**curl -X GET 'https://<Invoke URL>/lambdaapi' -H 'Content-Type: application/json' -H 'Authorization:<access_token>'**
 
   % curl -X GET 'https://h7l9qseapd.execute-api.us-east-1.amazonaws.com/Dev/lambdaapi' -H 'Content-Type: application/json' -H 'Authorization:eyJraWQiOiI3ak5xZ0hoZ3JCK0F3RGZcLzZ0bEdhZEJUTUhvbEJHdktNdXBqaTBBY2dRZz0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI0amg5dG9wcDQ1czFmYzhuNGc2NHNodGxzMiIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoibGFtYmRhQXBpXC9jcmVhdGVfbGFtYmRhIGxhbWJkYUFwaVwvcmVhZF9sYW1iZGEgbGFtYmRhQXBpXC9kZWxldGVfbGFtYmRhIiwiYXV0aF90aW1lIjoxNjUxNTY1NDk3LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9CUmFoeFBmSE0iLCJleHAiOjE2NTE1NjkwOTcsImlhdCI6MTY1MTU2NTQ5NywidmVyc2lvbiI6MiwianRpIjoiNTNhMGRhNjgtZWRkZC00MzBiLWFkNWItNTBkYmU5ZjAyM2QyIiwiY2xpZW50X2lkIjoiNGpoOXRvcHA0NXMxZmM4bjRnNjRzaHRsczIifQ.AtLo5xlxNx8C0vquzUMDPfGvivXIhfKOM3QFEf8PWb-phX2q86mxQMMHyYQ95gtEdb0Qdx7FrEc5zRdg94urvLyDprvlJospEa8x7UvGWO_9iJ-hp54QQFW4ySLjX3DMUQPUD7HXKeR9ycQmOyrBAA_EGHs1hEq2Vwc51X3FNTPbacMjyWMq7pMxtzwCKkt2Q9i6oOx85d7qxxJsJ_cAoYm1U_bGXO1jCYXUCBQsK7DcnDAKp4PSYMvvh2D4Q4vXx1WGOpLeb-RQyqeQNqMXUotbkxJV_TwMsnkCucHMukEaLBXKKaLiOdMawXd0WxbTV9P-dJOVUhZxzBvAnMWhGw'                             

   {"statusCode": 200, "body": "\"Hello from Get Lambda!\""}%                                                      
