# KBID 46 - SQLI

## Running the app nodeJs

First make sure nodejs and npm are installed on your host machine.
After installation, we go to the folder of the lab we want to practice.
"i.e /skf-labs/XSS, /skf-labs/RFI/" and run the following commands:

```
$ npm install
```

```
$ npm start
```

{% hint style="success" %}
Now that the app is running let's go hacking!
{% endhint %}

## Reconnaissance

### Step1

The first step is to identify parameters which could be potentially used in an SQL query to communicate with the underlying database. In this example we find that the "/home" method grabs data by pageID and displays the content.

![](../../.gitbook/assets/nodejs/SQLI/1.png)

```text
http://localhost:5000/home/1
```

### Step2

Now let's see if we can create an error by injecting a single quote

![](../../.gitbook/assets/nodejs/SQLI/2.png)

```text
http://localhost:5000/home/1'
```

By doing so the SQL query syntax is now faulty. This is due to the fact that the user supplied input is being directly concatenated into the SQL query.

```javascript
db.get(
  "SELECT pageId, title, content FROM pages WHERE pageId=" + req.params.pageId
);
```

### Step3

Now we can also use logical operators to determine whether we can actually manipulate the SQL statements.

We start with a logical operator which is false \(and 1=2\). The expected behaviour for injecting a false logical operator would be an error or 404.

![](../../.gitbook/assets/nodejs/SQLI/3.png)

```text
http://localhost:5000/home/1 and 1=2
```

After that we inject a logical operator which is true \(and 1=1\). This should result in the application run as intended without errors.

![](../../.gitbook/assets/nodejs/SQLI/4.png)

```text
http://localhost:5000/home/1 and 1=1
```

## Exploitation

Now that we know that the application is vulnerable for SQL injections we are going to use this vulnerability to read sensitive information from the database. This process could be automated with tools such as SQLMAP. However, for this example let's try to exploit the SQL injection manually.

### Step1

The UNION operator is used in SQL injections to join a query, purposely forged to the original query. This allows to obtain the values of columns of other tables. First we need to determine the number of columns used by the original query. We can do this by trial and error.

![](../../.gitbook/assets/nodejs/SQLI/5.png)

```text
http://localhost:5000/home/1 union select 1
```

This query results in an error, this is due to the fact that the original query started with 3 columns namely  
\* pageId  
\* title  
\* content

![](../../.gitbook/assets/nodejs/SQLI/6.png)

![](../../.gitbook/assets/nodejs/SQLI/7.png)

```text
http://localhost:5000/home/1 union select 1,2,3
```

Notice how "title" and "content" became placeholders for data we want to retrieve from the database

### Step 2

Now that we determined the number of columns we need to take an educated guess for the table we want to steal sensitive information from. Again we can see if we try to query a non existent table we get an error. For a correct table we see the application function as intended.

![](../../.gitbook/assets/nodejs/SQLI/8.png)

```text
http://localhost:5000/home/1 union select 1,2,3 from user
```

![](../../.gitbook/assets/nodejs/SQLI/9.png)

```text
http://localhost:5000/home/1 union select 1,2,3 from users
```

![](../../.gitbook/assets/nodejs/SQLI/10.png)

```text
http://localhost:5000/home/1 union select 1,username,password from users
```

## Mitigation

SQL Injection can be prevented by following the methods described below:

Primary Defenses:

First step: White-list Input Validation\
Second step: Use of Prepared Statements (Parameterized Queries)\

Additional Defenses:

Also: Enforcing Least Privilege\
Also: Performing Allow-list Input Validation as a Secondary Defense

In this case, we have presented a SQLi code fix by using parameterized queries (also known as prepared statements) instead of string concatenation within the query.

The following code is vulnerable to SQL injection as the user input is directly concatenated into query without any form of validation:
PATH:/SQLI/models/sqlimodel.py

```javascript
db.get(
  "SELECT pageId, title, content FROM pages WHERE pageId=" + req.params.pageId
);
```

This code can be easily rewritten in a way that prevent the user input from interfering with the query structure:

```javascript
db.get("SELECT pageId, title, content FROM pages WHERE pageId=?", [
  req.params.pageId,
]);
```

Can you try to implement input validation or escaping?

## Additional sources

Please refer to the OWASP testing guide for a full complete description about SQL injection with all the edge cases over different platforms!

{% embed url="https://owasp.org/www-community/attacks/SQL_Injection" %}

SQLite Reference

{% embed url="https://www.sqlitetutorial.net/sqlite-nodejs/query/" %}
