Here's a `README.md` file for your project:

```markdown
# Student Data Management with AWS Lambda and DynamoDB

This project demonstrates how to save and retrieve student data using AWS Lambda and DynamoDB, along with a simple HTML interface to interact with the data. The system allows users to:

- Save student information (Student ID, Name, Class, Age)
- View all student records

## Architecture Overview

1. **Frontend (HTML, JavaScript)**:
    - A simple HTML interface that collects student data from the user and displays the list of all students.
    - Uses **AJAX** to make API requests to AWS Lambda.

2. **Backend (AWS Lambda + DynamoDB)**:
    - AWS Lambda handles the logic for saving and retrieving student data.
    - **DynamoDB** is used to store the student records.

## Technologies Used

- **AWS Lambda**: To run the serverless backend code for saving and retrieving data.
- **AWS DynamoDB**: A NoSQL database used for storing student data.
- **HTML5**: For creating the web interface.
- **JavaScript (AJAX & jQuery)**: To interact with the Lambda API endpoints.
- **Boto3**: AWS SDK for Python, used in Lambda for DynamoDB interactions.

## Lambda Functions

### 1. Save Student Data

The Lambda function `lambda_handler` saves the student data in DynamoDB when triggered by an API call.

```python
import json
import boto3

def lambda_handler(event, context):
    # Initialize DynamoDB resource
    dynamodb = boto3.resource('dynamodb', region_name='us-east-2')
    table = dynamodb.Table('studentData')

    # Extract student data from event
    student_id = event['studentid']
    name = event['name']
    student_class = event['class']
    age = event['age']
    
    # Save data to DynamoDB table
    response = table.put_item(
        Item={
            'studentid': student_id,
            'name': name,
            'class': student_class,
            'age': age
        }
    )
    
    # Return success message
    return {
        'statusCode': 200,
        'body': json.dumps('Student data saved successfully!')
    }
```

### 2. Retrieve All Student Data

This Lambda function scans the `studentData` table to retrieve all stored student data.

```python
import json
import boto3

def lambda_handler(event, context):
    # Initialize DynamoDB resource
    dynamodb = boto3.resource('dynamodb', region_name='us-east-2')
    table = dynamodb.Table('studentData')

    # Scan DynamoDB table for all student data
    response = table.scan()
    data = response['Items']

    # Handle pagination if there are more items
    while 'LastEvaluatedKey' in response:
        response = table.scan(ExclusiveStartKey=response['LastEvaluatedKey'])
        data.extend(response['Items'])

    # Return all student data
    return data
```

## Frontend (HTML & JavaScript)

### HTML Interface

This is a simple form to allow users to input student data and view a list of all students stored in DynamoDB.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Student Data</title>
    <style>
        /* Styling code for the page */
    </style>
</head>
<body>
    <div class="container">
        <h1>Save and View Student Data</h1>
        <label for="studentid">Student ID:</label><br>
        <input type="text" name="studentid" id="studentid"><br>
        
        <label for="name">Name:</label><br>
        <input type="text" name="name" id="name"><br>
        
        <label for="class">Class:</label><br>
        <input type="text" name="class" id="class"><br>
        
        <label for="age">Age:</label><br>
        <input type="text" name="age" id="age"><br>
        
        <br>
        <input type="submit" id="savestudent" value="Save Student Data">
        <p id="studentSaved"></p>
        
        <br>
        <input type="submit" id="getstudents" value="View all Students">
        <br><br>
        <div id="showStudents">
            <table id="studentTable">
                <thead>
                    <tr>
                        <th>Student ID</th>
                        <th>Name</th>
                        <th>Class</th>
                        <th>Age</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Student data will be displayed here -->
                </tbody>
            </table>
        </div>
    </div>

    <script src="scripts.js"></script>
    <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.6.0/jquery.min.js"></script>
</body>
</html>
```

### JavaScript (AJAX Calls)

This script handles AJAX requests for saving student data and retrieving the list of all students.

```javascript
// Add your API endpoint here
var API_ENDPOINT = "API_ENDPOINT_PASTE_HERE";

// AJAX POST request to save student data
document.getElementById("savestudent").onclick = function() {
    var inputData = {
        "studentid": $('#studentid').val(),
        "name": $('#name').val(),
        "class": $('#class').val(),
        "age": $('#age').val()
    };
    $.ajax({
        url: API_ENDPOINT,
        type: 'POST',
        data: JSON.stringify(inputData),
        contentType: 'application/json; charset=utf-8',
        success: function(response) {
            document.getElementById("studentSaved").innerHTML = "Student Data Saved!";
        },
        error: function() {
            alert("Error saving student data.");
        }
    });
};

// AJAX GET request to retrieve all students
document.getElementById("getstudents").onclick = function() {
    $.ajax({
        url: API_ENDPOINT,
        type: 'GET',
        contentType: 'application/json; charset=utf-8',
        success: function(response) {
            $('#studentTable tr').slice(1).remove();
            jQuery.each(response, function(i, data) {
                $("#studentTable").append("<tr> \
                    <td>" + data['studentid'] + "</td> \
                    <td>" + data['name'] + "</td> \
                    <td>" + data['class'] + "</td> \
                    <td>" + data['age'] + "</td> \
                    </tr>");
            });
        },
        error: function() {
            alert("Error retrieving student data.");
        }
    });
}
```

## Setting Up

### Prerequisites

1. **AWS Account**: You will need an AWS account to set up Lambda and DynamoDB.
2. **API Gateway**: To expose your Lambda function via HTTP endpoints.
3. **IAM Role**: The Lambda function needs an IAM role with appropriate permissions to access DynamoDB.
4. **JavaScript**: jQuery should be included for making AJAX calls.

### Steps to Deploy

1. **Set up DynamoDB**:
   - Create a table named `studentData` in DynamoDB with `studentid` as the primary key.

2. **Create the Lambda Functions**:
   - Create two separate Lambda functions: one for saving student data (`lambda_handler` for `PUT` requests) and another for retrieving student data (`lambda_handler` for `GET` requests).

3. **Set Up API Gateway**:
   - Create an API using API Gateway to trigger Lambda functions.
   - Link the `POST` method to the Lambda function for saving data, and the `GET` method to the Lambda function for retrieving data.

4. **Deploy Frontend**:
   - Upload your HTML file and the JavaScript file to your web server or use an S3 bucket to host the frontend.

5. **Update API Endpoint**:
   - Replace `API_ENDPOINT_PASTE_HERE` with the actual endpoint URL provided by your API Gateway.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

This `README.md` file explains the setup, functionality, and deployment process for your project. You can adjust the details depending on the specific configurations or additional tools you use!
