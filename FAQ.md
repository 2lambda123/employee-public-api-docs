# API - Frequently asked questions and answers

On this page you will find some of the more common Q&A’s, the target demographic is developers integrating with the Employee API.

## Updating Employee or Position

The modification of data that is already in the service is done by overwriting the document and performed with a HTTP PUT request. The HTTP PUT request **overwrites** a representation of the target resource with the request payload. If update data includes oject value would not be set, that resource will be deleted.

- Adding new data ensures that historical values are preserved during the process and is critical for correct Employee salary and tax calculation and reporting to government from Visma.net HRM Payroll service. Updating data will always include changes to a [timeline object](https://github.com/visma-net/employee-public-api-docs/blob/main/FAQ.md#what-are-ojects-that-support-timelines).
- Overwriting data is intended for correcting faulty data and need to do that will occur rarely. When data is overwritten, timeline values are not retained and will be updated as well. **Used with caution**. Historical (timeline object values) data can also be overwritten by providing an updated timeline. For example a new salary date was set incorreclty starting with 1st of February in stead of 1st of January. This change would trigger recalculation as Employee's wage run was done with a wrong salary amount.

Examples:

- Correct flow to add a new salary: Get current position document, provide new amount for monthly salary, setting dates for new and old salary values, that takes effect starting with 1st of March, 2022.

  1. GET /v1/employees/{``employeeID``}}/positions/{``positionID``}

      ```json
          ...
          "salaryInformation": [
              {
                  "salaryType": "Period",
                  "monthlySalary": "1000" //When activeStart and activeEnd are not displayed, it means that this is initial value and is in effect at this point of time
              }
          ],
          ...
      ```

  1. Modify request payload and prepare for PUT operation. End date needs to be set to current salary. Start date needs to be set for new salary.

      ```json
          ...
          "salaryInformation": [
              {
                  "salaryType": "Period",
                  "monthlySalary": "10000",
                  "activeEnd": "2022-02-28" //Set last day when this salary whould be in effect
              },
              {
                  "salaryType": "Period",
                  "monthlySalary": "12300",
                  "activeStart": "2022-03-01" //Set when new salary should come in effect
              }
          ],
          ...
      ```

  1. PUT constructed payload and [check for job execution status](https://github.com/visma-net/employee-public-api-docs/blob/main/FAQ.md#working-with-status-code-202-and-jobs).

- Overwriting information in a scenario where faulty data has been provided. For example, wrong data has been provided in sample/payload above: new salary amount is incorrect. This request will overwrite target resource with the request payload.

  1. GET /v1/employees/{``employeeID``}}/positions/{``positionID``}

      ```json
          ...
          "salaryInformation": [
                {
                    "salaryType": "Period",
                    "monthlySalary": "10000",
                    "activeEnd": "2022-02-28"
                },
                {
                    "salaryType": "Period",
                    "monthlySalary": "12300",
                    "activeStart": "2022-03-01"
                }
            ],
          ...
      ```

  1. Modify request payload and prepare for PUT operation

      ```json
          ...
          "salaryInformation": [
              {
                  "salaryType": "Period",
                  "monthlySalary": "10000",
                  "activeEnd": "2022-02-28"
              },
              {
                  "salaryType": "Period",
                  "monthlySalary": "20000", //Set correct salary amount
                  "activeStart": "2022-03-01" 
              }
          ],
          ...
      ```

  1. PUT constructed payload and [check for job execution status](https://github.com/visma-net/employee-public-api-docs/blob/main/FAQ.md#working-with-status-code-202-and-jobs).

## Working with status code 202 and Jobs

When a POST or PUT request is accepted, a job is created for background processing. In response header you will find "Location" attribute that contains the uri for the job (path and ID). Until the job has not executed, there is no artifact (employee or position) created.
Query the location uri to get job details: execution status, message in case of an error and other information.
Contents of the response:

- *status*: status of job execution.
- *artifactId*: ID for the object that is created or updated. In case of succesfull job execution, ID of the object created or updated will be listed here.
- *artifactType*: type of the object that is created or updated.
- *validationResults*: validation result in case there is an issue with the payload data or operation that is being executed. Warning and/or errors with descritpion will be displayed here.There can be one or multiple validation results displayed. 
- other information

Example: GET /employees/jobs/xxxxxxxx-80d5-46eb-yyyy-4bc4cea5xxxx

```json
{
    "correlationId": "xxxxxxxx-80d5-46eb-a522-4bc4cea5xxxx",
    "clientId": "xxxx",
    "id": "xxxxxxxx-80d5-46eb-yyyy-4bc4cea5xxxx",
    "tenantId": "xxxx",
    "created": "2021-12-16T11:25:09.605+00:00",
    "artifactId": "xxxxxxxx-yyyy-46eb-a522-4bc4cea5xxxx",
    "artifactType": "Employee",
    "status": "Succeeded",
    "statusCode": 4,
    "action": "Create",
    "lastChange": "2021-12-16T11:25:13.593+00:00",
    "validationResults": []
}
```

## Minimal set of mandatory fields for POST /employees/withPosition

Mandatory fields can vary based on the data that you choose to provide when creting an employee. 
You will always have to provide data on top level of Employee profile for the following properties:

```json
  {
  "number": "",
  "personNames": [
    {
      "firstName": "",
      "lastName": ""
    }
  ],
  "personIds": [],
  "dateOfBirth": "",
  "address": {
    "countryCode": ""
  },
  "salaryPaymentMethod": {},
  "positions": [
    {
      "typeOfPosition": "",
      "employmentForm": [],
      "typeOfWork": [],
      "workTimeAgreement": [],
      "partTimeFactors": [],
      "salaryInformation": [],
      "taxUnitIds": [],
      "activeStart": ""
    }
  ]
}
  ```

### Sample payload with minimal set of mandatory fields used for POST /employees/withPosition

```json
{
  "number": "1230",
  "personNames": [
    {
      "firstName": "Robert",
      "lastName": "Trully"
    }
  ],
  "personIds": [{
      "typeOfId": "DNumber",
      "id": "67024795818"
    }],
  "dateOfBirth": "1983-03-14",
  "address": {
    "countryCode": "NO"
  },
  "salaryPaymentMethod": {"paymentType": "cash"},
  "positions": [
    {
      "typeOfPosition": "Ordinary",
      "employmentForm": [{
          "value": "fast"
        }],
      "typeOfWork": [{
          "value": "1233101"
        }],
      "workTimeAgreement": [{
          "value": "0387f476-2471-4816-8b94-cdabb8fe4c21"
        }],
      "partTimeFactors": [{
          "value": 100
        }],
      "salaryInformation": [{
          "salaryType": "Period",
          "monthlySalary": "1000"
        }],
      "taxUnitIds": [{
          "value": "2"
        }],
      "activeStart": "2020-02-02"
    }
  ]
}
```

## What are ojects that support timelines?

Visma.net Employee API supports objects that can have multiple values in different time frames, called timelines.  
Timeline is an option to manage historic, current and future values for various supported objects or values.</br>
When retreiving data, without any filters, you will always see:

- historic values - values that have been defined in the past (in case there have been changes)</li>
- active (current) values in effect today</li>
- future values - values that will be applied in the future (in case have been defined)</li></br>
</br>
activeStart and activeEnd are date items expressed as string values.
  
  - activeStart - is a date value indicating when value takes effect/ becomes active. Null means infinity. From what date this configuration of the value is active.</li>
  - activeEnd - is a date value indicating when value losses effect or becomes inactive. Null means infinity. To what date this configuration of the value is active</li>
  </br>
  In order to correctly update a value with timeline support, activeEnd needs to be defined for current active value. New value has to have activeStart defined.  

Examples of timeline usage

1. Oject timeline upon new employee creation, when object timeline values are null. This means that the value is active since the begging of time and has no end date

    ```json
    "salaryInformation": [
      {
        "salaryType": "Period",
        "monthlySalary": "4000"
      }
    ]
    ```

2. Assuming that it is March, 2021, employee with current salary (valid untill April 30th, 2021) and a new salary that will become in effect starting with May 1st, 2021

    ```json
    "salaryInformation": [
      {
        "salaryType": "Period",
         "monthlySalary": "3000", //Historic value that was active untill December 13th, 2020
         "activeEnd": "2020-12-13"
      },
      {
        "salaryType": "Period",
        "monthlySalary": "4000", //Currently active timeline value assuming that it is March, 2021
        "activeStart": "2020-12-14",  
        "activeEnd": "2021-04-30" //Current value will expire in the end of April 
      },
      {
        "salaryType": "Period",
        "monthlySalary": "5000", //Future timeline value
        "activeStart": "2021-05-01"
      }
    ]
    ```

Some objects can have overlapping periods and/or can gaps. 
Examples:

  1. Pensions is an exception where there can be multiple pensions defined for an employee, where timeline can overlap. 
  2. Postitions is an object that can have gaps in timeline. When position ends, employee's employment is terminated. Employee later can be rehired which creates a gap in a timeline.
  
---
