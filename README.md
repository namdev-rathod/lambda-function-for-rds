# lambda-function-for-rds
 - Lambda Function To Connect MySQL RDS & Run Queries
 - Write lambda function in nodejs 18
 - Use AWS-SDK library
 - Password should stored in secret manager

```
"use strict";
const AWS = require("aws-sdk");
const mysql = require('mysql2/promise');

exports.handler = async (event, context) => {
  // Initialize the AWS SDK and create an RDS client
  const secretsManager = new AWS.SecretsManager();
  const secretName = "dbpassword"; // Replace with your Secret Manager secret name
  const rds = new AWS.RDS();

  try {
    // Get RDS MySQL credentials from Secrets Manager
    const secretValue = await secretsManager.getSecretValue({ SecretId: secretName }).promise();
    const secretString = secretValue.SecretString;
    const secretData = JSON.parse(secretString);

    // Create a MySQL connection
    const connection = await mysql.createConnection({
      host: secretData.host,
      user: secretData.username,
      password: secretData.password,
      database: secretData.dbname
    });

    // Fetch record details from an RDS MySQL table
    const [rows, fields] = await connection.execute('drop database demoawsdb');

    // Process or return the record details
    console.log('Fetched records:', rows);

    connection.end();

    return {
      statusCode: 200,
      body: JSON.stringify(rows)
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Error fetching records' })
    };
  }
};

