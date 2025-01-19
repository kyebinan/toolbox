## API Penetration Testing Cheatsheet

### API Documentation

- **Human-readable Documentation**: Contains detailed explanations, examples, and usage scenarios.
- **Machine-readable Documentation**: Written in structured formats like JSON or XML for automation.

### Discovering API Documentation

- Use Burp Scanner to crawl applications for endpoints like:
  - `/api`
  - `/swagger/index.html`
  - `/openapi.json`

### Interacting with API Endpoints

- **Identifying Endpoints**: Look for patterns in URL structures like `/api/`.
- **Supported HTTP Methods**: Test methods like `GET`, `POST`, `DELETE` to discover functionality.
- **Supported Content Types**: Modify `Content-Type` headers to trigger different behaviors (e.g., JSON vs. XML).

### Using Burp Suite Tools

- **Burp Repeater**: Interact manually with API endpoints.
- **Burp Intruder**: Automate testing by cycling through HTTP methods or payloads.
- **Burp Scanner**: Crawls and audits OpenAPI documentation and endpoints.

### Finding Hidden Endpoints and Parameters

- **Intruder for Hidden Endpoints**: Use wordlists to discover endpoints similar to known paths.
- **Parameter Discovery**: Use tools like Param Miner to guess and test hidden parameters.
- **Content Discovery**: Identify hidden content and parameters not linked from visible browsing.


### Mass assignment vulnerabilities

Mass assignment (also known as auto-binding) can inadvertently create hidden parameters. It occurs when software frameworks automatically bind request parameters to fields on an internal object. Mass assignment may therefore result in the application supporting parameters that were never intended to be processed by the developer.

Identifying hidden parameters 

Since mass assignment creates parameters from object fields, you can often identify these hidden parameters by manually examining objects returned by the API.

For example, consider a PATCH /api/users/ request, which enables users to update their username and email, and includes the following JSON:

{
    "username": "wiener",
    "email": "wiener@example.com",
}
A concurrent GET /api/users/123 request returns the following JSON:

{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
This may indicate that the hidden id and isAdmin parameters are bound to the internal user object, alongside the updated username and email parameters.


Testing mass assignment vulnerabilities

To test whether you can modify the enumerated isAdmin parameter value, add it to the PATCH request:

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": false,
}
In addition, send a PATCH request with an invalid isAdmin parameter value:

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": "foo",
}
If the application behaves differently, this may suggest that the invalid value impacts the query logic, but the valid value doesn't. This may indicate that the parameter can be successfully updated by the user.

You can then send a PATCH request with the isAdmin parameter value set to true, to try and exploit the vulnerability:

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true,
}
If the isAdmin value in the request is bound to the user object without adequate validation and sanitization, the user wiener may be incorrectly granted admin privileges. To determine whether this is the case, browse the application as wiener to see whether you can access admin functionality.

### Server-side parameter pollution 

Some systems contain internal APIs that aren't directly accessible from the internet. Server-side parameter pollution occurs when a website embeds user input in a server-side request to an internal API without adequate encoding. This means that an attacker may be able to manipulate or inject parameters, which may enable them to, for example:

Override existing parameters.
Modify the application behavior.
Access unauthorized data.
You can test any user input for any kind of parameter pollution. For example, query parameters, form fields, headers, and URL path parameters may all be vulnerable.



Note
This vulnerability is sometimes called HTTP parameter pollution. However, this term is also used to refer to a web application firewall (WAF) bypass technique. To avoid confusion, in this topic we'll only refer to server-side parameter pollution.

In addition, despite the similar name, this vulnerability class has very little in common with server-side prototype pollution.


### Testing for server-side parameter pollution in the query string 

To test for server-side parameter pollution in the query string, place query syntax characters like #, &, and = in your input and observe how the application responds.

Consider a vulnerable application taht enables you to search for others users based on their username. When you search for a user, your browser makes the following request:

GET /userSearch?name=peter&back=/home

To retrieve user information, the server queries an internal API with the following request:

GET /users/search?name=peter&publicProfile=true

### Truncating query strings 

You can use a URL-encoded # character to attempt to truncate the server-side request. To help you interpret the response, you could also add a string after the # character.


For example, you could modify the query string to the following:

GET /userSearch?name=peter%23foo&back=/home
The front-end will try to access the following URL:

GET /users/search?name=peter#foo&publicProfile=true


Note
It's essential that you URL-encode the # character. Otherwise the front-end application will interpret it as a fragment identifier and it won't be passed to the internal API.

Review the response for clues about wheter the query has been truncated. For example, if the reponse returns thee user peter, the server-side query may have been truncated. If an Invalid name error message is returned, the application may have treated foo as part of the username. This suggests that the server-side request may not have been truncated.

If you're able to truncate the server-side request, his removes the requirement for the publicProfile field to be set to true. You may be able to exploit this to return non-public user profiles.

### Injecting invalid Parameters

You can use an URL-encoded & character to attempt to add a second parameter to the server-side request.

For example, you could modify the query string to the following:

GET /userSearch?name=peter%26foo=xyz&back=/home
This results in the following server-side request to the internal API:

GET /users/search?name=peter&foo=xyz&publicProfile=true
Review the response for clues about how the additional parameter is parsed. For example, if the response is unchanged this may indicate that the parameter was successfully injected but ignored by the application.

To build up a more complete picture, you'll need to test further.

### Injecting valid parameters 

If you're able to modify the query string, you can then attempt to add a second valid parameter to the server-side request.

Related pages
For information on how to identify parameters that you can inject into the query string, see the Finding hidden parameters section.

For example, if you've identified the email parameter, you could add it to the query string as follows:

GET /userSearch?name=peter%26email=foo&back=/home
This results in the following server-side request to the internal API:

GET /users/search?name=peter&email=foo&publicProfile=true
Review the response for clues about how the additional parameter is parsed.

### Overriding existing parameters 

To confirm wheter the application is vulnerable to server-side parameter pollution, you could try to Override the original parameter. Do this by injection a second parameter with the same name.

For example, you could modify the query string to the following:

GET /userSearch?name=peter%26name=carlos&back=/home
This results in the following server-side request to the internal API:

GET /users/search?name=peter&name=carlos&publicProfile=true
The internal API interprets two name parameters. The impact of this depends on how the application processes the second parameter. This varies across different web technologies. For example:

PHP parses the last parameter only. This would result in a user search for carlos.
ASP.NET combines both parameters. This would result in a user search for peter,carlos, which might result in an Invalid username error message.
Node.js / express parses the first parameter only. This would result in a user search for peter, giving an unchanged result.
If you're able to override the original parameter, you may be able to conduct an exploit. For example, you could add name=administrator to the request. This may enable you to log in as the administrator user.

### Testing for service-side parameter pollution in REST paths 

A RESTful API may place parameter names and values in the URL path, rather than the query string. For example, consider the following path:

/api/users/123
The URL path might be broken down as follows:

/api is the root API endpoint.
/users represents a resource, in this case users.
/123represents a parameter, here an identifier for the specific user.
Consider an application that enables you to edit user profiles based on their username. Requests are sent to the following endpoint:

GET /edit_profile.php?name=peter
This results in the following server-side request:

GET /api/private/users/peter
An attacker may be able to manipulate server-side URL path parameters to exploit the API. To test for this vulnerability, add path traversal sequences to modify parameters and observe how the application responds.

You could submit URL-encoded peter/../admin as the value of the name parameter:

GET /edit_profile.php?name=peter%2f..%2fadmin
This may result in the following server-side request:

GET /api/private/users/peter/../admin
If the server-side client or back-end API normalize this path, it may be resolved to /api/private/users/admin.



