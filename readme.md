1) A way to create /host a webpage 
2) A way to invoke the math function 
3) A way to do some math 
4) Somewhere to Store/return the math result
5) A way to handle permission 





1) create a index.html file  copy and paste to your index.html 
  ************************************************************

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
</head>

<body>
    To the Power of Math!
</body>
</html>


save and create a zip file from your index.html 

we are going to use applify to deploy index.html

On your aws console and go to your AWS Amplify go to create a Host your web app

Projet we dont have a git provider so we just select Deploy with out git provider 
and click on continue . 

Give your App a name "Gabriel App" and Environment name "Dev Gabby" click on drag and drop 
uplaod / drag the Zip index.html we created click on save and Deploy 

After the Deployment is succesful us will see ypur app url link copy and paste it on your 
browser . now we now have a user webpage that a user can navigate to 

A way to do some math 
*********************
we are going to use a lamda function = code that runs upon (saverlessly) some trigger
Now we write some phyton code and use the python code library

On your aws console and go to your lamda clickon create a lamda function click on 
Autor from scratch give your fuction a name "PowerOfMathFunction" for runtime 
go for python 3.9 and leave everything the same and click on create Function

update the code source with this 

# import the JSON utility package
import json
# import the Python math library
import math

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):

# extract the two numbers from the Lambda service's event object
    mathResult = math.pow(int(event['base']), int(event['exponent']))

    # return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }

save and deploy the code . now let's test to make sure this function as we expect so click the little drop down arrow on the right test button and set up a test event this basically lets us pass in some test data to make sure the function is working 

Create a new event name  "PowerOfMathTestevent"
Event sharing  private
in the stater code "Event JSON" we only have two vaule to pass in. one of is going to be base & exponent 

{
  "base": 2,
  "exponent": 3
}




"2 raised to the power of 3"  =  to 8.

click on save 

Now we just finish configuring the test event now let run test click on the test button . we will get a status code of 200 and result is 8 

Our lamda function is working 

now we have create a simple html page hosted on  Amplify and a lamda function to do some basic math

A way to invoke the math function
**********************************
Now we are going to build a Api Gateway . 
API Gateway allows developers to create and deploy APIs that act as a front door for applications to access data, business logic, or functionality from backend services.

Api gateway is a core servive in AWS that we can use to build our own application programming interface "http or websocket api " it's the perfect way to invoke a lamda function 

On your aws console navigate to api gateway 
click on create api. we are going to be using rest api click on the rest api build button 

leave the frist two options as they are so this will be REST and a NEW API and give it a name "PowerOfmathApi" leave everything else the same and click on create API
now we have sort of empty API which has no methods that somebody will call. on the API we juct create select Resources . you have to get the backslash selected and on the action menu create method the type of method will be a post "remmber the user is sending in some data for those two numbers " click on the check mark for integration type . we are going to user the lamda function and then the lamda funtion name "PowerofMathFunction" and click on save 

Enable cores or cross origin resource sharing . select the post go to the Action menu enable CORS and "Basically what this dose is allows a web application running or domain  to be able to access resources on a diffrent orgin or domain because our web application is running on one domain and amplify ourlamda function is going to be running in another we need to be able to work across those domain or origins" click on enable CORS and click on  yes

Deploy API and test it on the Action menu click on deploy API and set up a new stage , stage name "DEV" and copy the api gateway url from your stage editor somewhere on your note pad "https://oghlni0wki.execute-api.us-east-1.amazonaws.com/dev" . now go to your api Resource click on post . you'll see a flow of melthod request, integration request, melthod response ,integration response. to test if is API gateway is working click on the blue button this will let us send whatever we want to test 
on the request button copy and past 
base 2 
exponet 3
then click on test 

Somewhere to Store/return the math result and A way to handle permission
************************************************************************
we are using dynamoDB (A key Vaule "NO SQL" Database)
we are going to give our lamda funtion permission to write to the Database 
Create a table in DynamoDB . Table name "powerOfMathDatabase" partition key "id" leave everthin else the same and create table 

Amazon Resource Name (ARN)
arn:aws:dynamodb:us-east-1:595851316359:table/PowerOfMathDatabase


go to the lamda we created and select configuration and clicck on permission click on the Execution role link it will open on a new tab now go to add permission and click on create in line policy click on json tab copy and paste this code to the jsob tab

{
"Version": "2012-10-17",
"Statement": [
    {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
            "dynamodb:PutItem",
            "dynamodb:DeleteItem",
            "dynamodb:GetItem",
            "dynamodb:Scan",
            "dynamodb:Query",
            "dynamodb:UpdateItem"
        ],
        "Resource": "YOUR-TABLE-ARN"
    }
    ]
}

click on review policy give your policy a name "powerofmathDBpolicy" and click on create policy 

update the lamda function to actually go right to the Database . on the execution tab copy and paste this code on function.py 

# import the JSON utility package
import json
# import the Python math library
import math

# import the AWS SDK (for Python the package name is boto3)
import boto3
# import two packages to help us with dates and date formatting
from time import gmtime, strftime

# create a DynamoDB object using the AWS SDK
dynamodb = boto3.resource('dynamodb')
# use the DynamoDB object to select our table
table = dynamodb.Table('PowerOfMathDatabase')
# store the current time in a human readable format in a variable
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):

# extract the two numbers from the Lambda service's event object
    mathResult = math.pow(int(event['base']), int(event['exponent']))

# write result and time to the DynamoDB table using the object we instantiated and save response in a variable
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime':now
            })

# return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }


save and Deploy click on click on test 
base 2 
exponet 3
click on test to check the result 
Go to Explore table item to show you what has been stored there . now you can see our result and time that it happen 
now connect Amplify go to index.html and update it with this code 

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
    <!-- Styling for the client UI -->
    <style>
    h1 {
        color: #FFFFFF;
        font-family: system-ui;
		margin-left: 20px;
        }
	body {
        background-color: #222629;
        }
    label {
        color: #86C232;
        font-family: system-ui;
        font-size: 20px;
        margin-left: 20px;
		margin-top: 20px;
        }
     button {
        background-color: #86C232;
		border-color: #86C232;
		color: #FFFFFF;
        font-family: system-ui;
        font-size: 20px;
		font-weight: bold;
        margin-left: 30px;
		margin-top: 20px;
		width: 140px;
        }
	 input {
        color: #222629;
        font-family: system-ui;
        font-size: 20px;
        margin-left: 10px;
		margin-top: 20px;
		width: 100px;
        }
    </style>
    <script>
        // callAPI function that takes the base and exponent numbers as parameters
        var callAPI = (base,exponent)=>{
            // instantiate a headers object
            var myHeaders = new Headers();
            // add content type header to object
            myHeaders.append("Content-Type", "application/json");
            // using built in JSON utility package turn object to string and store in a variable
            var raw = JSON.stringify({"base":base,"exponent":exponent});
            // create a JSON object with parameters for API call and store in a variable
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                body: raw,
                redirect: 'follow'
            };
            // make API call with parameters and use promises to get response
            fetch("YOUR API GATEWAY ENDPOINT", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
        }
    </script>
</head>
<body>
    <h1>TO THE POWER OF MATH!</h1>
	<form>
        <label>Base number:</label>
        <input type="text" id="base">
        <label>...to the power of:</label>
        <input type="text" id="exponent">
        <!-- set button onClick method to call function we defined passing input values as parameters -->
        <button type="button" onclick="callAPI(document.getElementById('base').value,document.getElementById('exponent').value)">CALCULATE</button>
    </form>
</body>
</html>

