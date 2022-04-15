# SCV-executeFlow-api
Modification to the InvokeSalesforceRestApiFunction Lambda function to allow direct invocation of Salesforce Flows via REST API


# Service Cloud Voice - Execute Flow via API

## Usage

**Assume Service Cloud Voice has already been configured, the InvokeSalesforceRestApiFunction Lambda function has already been configured for authentication with Salesforce and modified to use the ‘executeFlow’ method**

### Salesforce

1. Create a new Salesforce Flow and activate it
2. For each input from Amazon Connect
    1. Create a ‘**New Resource**’
    2. **Resource Type = Variable**
    3. **Data Type = Text**
    4. **Default Value** (optional)
    5. **Available for Input (CHECKED) <----Critical**
3. For each Flow output you want to return to Amazon Connect
    1. Create a New Resource variable just like above, but...
    2. **Available for Output (CHECKED) <----Critical**
4. Note the following values you’ll need for Amazon Connect
    1. Flow API Name
    2. Each variable’s API name and if it’s input or output

### Amazon Connect

1. Open or create a Contact Flow
2. Drag an ‘Invoke AWS Lambda Function’ onto the canvas
3. Double click the block to edit
4. Under ‘Select a function’, select **InvokeSalesforceRestApiFunction**
5. Under ‘Function input parameters’, click ‘Add a parameter’
6. Select ‘Use Text’, Destination Key = **methodName**, Value = **executeFlow**
7. Add a 2nd parameter
8. Select ‘Use Text’, Destination Key = **flowApiName**, Value = **(*the API value of your Salesforce Flow)***
9. Add a additional parameters for EACH value you want to pass into the Salesforce Flow. Note: these must EXACTLY match the flow input variable API names.
    1. For example, passing the caller’s phone number into the flow you might use 
    2. Select ‘Use Attribute’, Destination Key = **contactPhone**,  Type = **System**, Attribute = **Customer Number**
10. To reference the Salesforce Flow output variables, use: `$.External.attributeName `where `AttributeName` is the attribute name/variable api name, or the key of the key-value pair returned from the function.

**NOTE:** Lambda external attributes are returned as key-value pairs _from the most recent invocation_ of an **Invoke AWS Lambda function** block. External attributes are overwritten with each invocation of the Lambda function.

https://docs.aws.amazon.com/connect/latest/adminguide/connect-attrib-list.html

## Code Changes to InvokeSalesforceRestApiFunction

### Key Modifications

* handler.js
    * new case for **executeFlow** methodName
        * invokes sfRestApi excuteFlow
* sfRestApi.js
    * new function **executeFlow(flowApiName, fieldValues)**
        * flowApiName = API Name of Salesforce function to call
        * fieldValues = pass in Amazon Connect parameters
        * returns collection of key value pairs from Salesforce Flow variables enabled for ‘output’
* utils.js
    * new function **getFlowInputFieldValuesFromConnectLambdaParams(params)**
        * Essentially copies getSObjectFieldValuesFromConnectLambdaParams(params) and modifies for Flow API input

### Untested

* Zero input variables
* Zero output results
* Output with nested values 
* Error handling and logging

### Probably Poor Coding

**handler.js > getFlowInputFieldValuesFromConnectLambdaParams**
This probably isn’t the best way to format the Flow API input requirements with JSON; but it works and JSON vs JS objects is hard?!!

```
157: const inputs = {"inputs": [fieldValues]};
```


