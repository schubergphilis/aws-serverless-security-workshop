# Module 4: Use SSL in-transit for your DB connections

Although we are using VPC and traffic is private within it, some regulations or compliance requirements might require encryption in transit. This encryption secures the data when communicating with the database. 

Go to *dbUtils.js* to add a new property to your database connection. Under the method ***getDbConfig***, within the resolve object (a JSON object), add a new line to the JSON:

```
    ssl: "Amazon RDS",

```
The resolve should be like this:

<details>
<summary><strong>If you haven't gone through AWS Secrets Manager step</strong></summary><p>

```javascript
			resolve({
			    ssl: "Amazon RDS",
			    host: host,
			    user: "admin",
			    password: "Corp123!",
			    database: "unicorn_customization",
			    multipleStatements: true
			});
```
</details>

<details>
<summary><strong>If you have gone through AWS Secrets Manager step</strong></summary><p>

```javascript
            client.getSecretValue({SecretId: secretName}, function (err, data) {
                if (err) {
                    console.error(err);
                    if (err.code === 'ResourceNotFoundException')
                        reject("The requested secret " + secretName + " was not found");
                    else if (err.code === 'InvalidRequestException')
                        reject("The request was invalid due to: " + err.message);
                    else if (err.code === 'InvalidParameterException')
                        reject("The request had invalid params: " + err.message);
                    else
                        reject(err.message);
                }
                else {
                    if (data.SecretString !== "") {
                        secret = data.SecretString;
                        resolve({
                            ssl: "Amazon RDS",
                            host: JSON.parse(secret).host,
                            user: JSON.parse(secret).username,
                            password: JSON.parse(secret).password,
                            database: "unicorn_customization",
                            multipleStatements: true
                        });
                    } else {
                        reject("Cannot parse DB credentials from secrets manager.");
                    }
                }
            });
```
</details>

Finally, deploy these changes:

```bash
cd ~/environment/aws-serverless-security-workshop/src
aws cloudformation package --output-template-file packaged.yaml --template-file template.yaml --s3-bucket $BUCKET --s3-prefix securityworkshop --region $REGION &&  aws cloudformation deploy --template-file packaged.yaml --stack-name CustomizeUnicorns --region $REGION --capabilities CAPABILITY_IAM --parameter-overrides InitResourceStack=Secure-Serverless
```

Once this is done, you should be able to connect to the database using SSL.

## Ensure SSL - Optional step

You can require SSL connections for specific users accounts\. For example, you can use the following statement, to require SSL connections on the user account `encrypted_user`\.


```
GRANT USAGE ON *.* TO 'encrypted_user'@'%' REQUIRE SSL;           
```
For more information on SSL connections with MySQL, go to the [MySQL documentation](https://dev.mysql.com/doc/refman/5.6/en/secure-connections.html)\.

To find the MySQL version of the Aurora database, go to the RDS console and find the **Engine version** under **Configuration** tab of the database cluster:

![](images/check-engine-version.png)

## Next step 
You have now further secured your data by enabling encryption in transit for your database connection! 

Return to the workshop [landing page](../../README.md) to pick another module.
