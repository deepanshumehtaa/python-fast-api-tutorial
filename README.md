# Tutorial - User Guide
# 1. Install FastAPI
    pip install "fastapi[all]"

    + Want to deploy your app to production
    pip install fastapi

    + install uvicorn to work as the server
    pip install "uvicorn[standard]"

# 2. Run the live server
    uvicorn main:app --reload
    
    + The command uvicorn main:app refers to:
    main: the file main.py (the Python "module").
    app: the object created inside of main.py with the line app = FastAPI().
    --reload: make the server restart after code changes. Only use for development.

# 3. Check it
    Open your browser at http://127.0.0.1:8000

# 4. Interactive API docs
    Now go to http://127.0.0.1:8000/docs
    You will see the automatic interactive API documentation (provided by Swagger UI)

# 5. Alternative API docs
    And now, go to http://127.0.0.1:8000/redoc.
    You will see the alternative automatic documentation (provided by ReDoc)

# 6. Query Parameters
    When you declare other function parameters that are not part of the path parameters, they are automatically interpreted as "query" parameters.

# 7. Request Body
    + When you need to send data from a client (let's say, a browser) to your API, you send it as a request body.
    + A request body is data sent by the client to your API. A response body is the data your API sends to the client.

# 8. Singular values in body
    The same way there is a Query and Path => FastAPI provides an equivalent Body.

# 9. Body - Fields
    + The same way you can declare additional validation and metadata in path operation function parameters with Query, Path and Body => Field.
    + Notice that Field is imported directly from pydantic, not from fastapi as are all the rest (Query, Path, Body, etc).

# 10. Extra Data Types
    + UUID:
    - A standard "Universally Unique Identifier", common as an ID in many databases and systems.
    - In requests and responses will be represented as a str.
    + datetime.datetime:
    - A Python datetime.datetime.
    - In requests and responses will be represented as a str in ISO 8601 format, like: 2008-09-15T15:53:00+05:00.
    + datetime.date:
    - Python datetime.date.
    - In requests and responses will be represented as a str in ISO 8601 format, like: 2008-09-15.
    + datetime.time:
    - A Python datetime.time.
    - In requests and responses will be represented as a str in ISO 8601 format, like: 14:23:55.003.
    + datetime.timedelta:
    - A Python datetime.timedelta.
    - In requests and responses will be represented as a float of total seconds.
    - Pydantic also allows representing it as a "ISO 8601 time diff encoding"
    + frozenset:
    - In requests and responses, treated the same as a set:
        . In requests, a list will be read, eliminating duplicates and converting it to a set.
        . In responses, the set will be converted to a list.. The generated schema will specify that the set values are unique (using JSON Schema's uniqueItems).
    + bytes:
    - Standard Python bytes.
    - In requests and responses will be treated as str.
    - The generated schema will specify that it's a str with binary "format".
    + Decimal:
    - Standard Python Decimal.
    - In requests and responses, handled the same as a float.

# 11. Cookie Parameters
    + You can define Cookie parameters the same way you define Query and Path parameters
    + Declare Cookie parameters: The first value is the default value, you can pass all the extra validation or annotation parameters

# 12. Response Model
    + You can declare the model used for the response with the parameter response_model in any of the path operations:
    @app.get()
    @app.post()
    @app.put()
    @app.delete()
    + Notice that response_model is a parameter of the "decorator" method (get, post, etc). Not of your path operation function, like all the parameters and body.

# 13. Extra Models
    + This is especially the case for user models, because:
    . The input model needs to be able to have a password.. . The output model should not have a password.
    . The database model would probably need to have a hashed password.

    + About **user_in.dict()
    - user_in is a Pydantic model of class UserIn.
    - Pydantic models have a .dict() method that returns a dict with the model's data.
    - So, if we create a Pydantic object user_in like:
        user_in = UserIn(username="john", password="secret", email="john.doe@example.com")
    and then we call:
        user_dict = user_in.dict()

    + UserInDB(**user_dict)
    - Would result in something equivalent to:
        UserInDB(
            username="john",
            password="secret",
            email="john.doe@example.com",
            full_name=None,
        )
    - Or more exactly, using user_dict directly, with whatever contents it might have in the future:
        UserInDB(
            username = user_dict["username"],
            password = user_dict["password"],
            email = user_dict["email"],
            full_name = user_dict["full_name"],
            hashed_password = hashed_password,
        )
    
    + Union or anyOf
    - You can declare a response to be the Union of two types, that means, that the response would be any of the two. => It will be defined in OpenAPI with anyOf.

# 14. Status Codes
    + 100 and above are for "Information". You rarely use them directly. Responses with these status codes cannot have a body.
    + 200 and above are for "Successful" responses. These are the ones you would use the most.
        . 200 is the default status code, which means everything was "OK".
        . Another example would be 201, "Created". It is commonly used after creating a new record in the database.
        . A special case is 204, "No Content". This response is used when there is no content to return to the client, and so the response must not have a body.
    + 300 and above are for "Redirection". Responses with these status codes may or may not have a body, except for 304, "Not Modified", which must not have one.
    + 400 and above are for "Client error" responses. These are the second type you would probably use the most.
        . An example is 404, for a "Not Found" response.
        . For generic errors from the client, you can just use 400.
    + 500 and above are for server errors. You almost never use them directly. When something goes wrong at some part in your application code, or server, it will automatically return one of these status codes.

# 15. Form Data
    When you need to receive form fields instead of JSON, you can use Form

# 16. Request Files
    To receive uploaded files, first install python-multipart.
        pip install python-multipart

    This is because uploaded files are sent as "form data".

    + Using UploadFile has several advantages over bytes:
        . It uses a "spooled" file:
            * A file stored in memory up to a maximum size limit, and after passing this limit => stored in disk.
        . work well for large files like images, videos, large binaries, etc. without consuming all the memory.
        . can get metadata from the uploaded file.
        . a file-like async interface.
        . exposes an actual Python SpooledTemporaryFile object that you can pass directly to other libraries that expect a file-like object.

# 17. Handling Errors
    + Use HTTPException
    + Add custom headers
    + Install Custom exception handlers
    + Override the default exception handlers

# 18. Path Operation Configuration
    + Response Status Code
    + Tags
    + Summary and description
    + Description from docstring
    + Response description
    + Deprecate a path operation

# 19. JSON Compatible Encoder
    + There are some cases to convert a data type (like a Pydantic model) to something compatible with JSON (like a dict, list, etc).
    + For example, if you need to store it in a database.
    + FastAPI provides a jsonable_encoder() function

# 20. Body - Updates
    + Update replacing with PUT
        . To update an item you can use the HTTP PUT operation.
        . You can use the jsonable_encoder to convert the input data to data that can be stored as JSON (e.g. with a NoSQL database). For example, converting datetime to str.
    
    + Partial updates with PATCH
        . PATCH is less commonly used and known than PUT.
        . And many teams use only PUT, even for partial updates.
        . You are free to use them however you want, FastAPI doesn't impose any restrictions.

# 21. Dependencies
    + "Dependency Injection" means, in programming, there is a way for your code (in this case, your path operation functions) to declare things that it requires to work and use: "dependencies".
    + That system (in this case FastAPI) will take care of doing whatever is needed to provide your code with those needed dependencies ("inject" the dependencies).
        . Have shared logic (the same code logic again and again).
        . Share database connections.
        . Enforce security, authentication, role requirements, etc.
        . And many other things...
    => All these, while minimizing code repetition.

# 22. Security
    + OAuth2: 
        . OAuth2 is a specification that defines several ways to handle authentication and authorization.
        . It is quite an extensive specification and covers several complex use cases.
        . It includes ways to authenticate using a "third party".
        . That's what all the systems with "login with Facebook, Google, Twitter, GitHub" use underneath.
    + OpenID Connect: 
        . OpenID Connect is another specification, based on OAuth2.
        . It just extends OAuth2 specifying some things that are relatively ambiguous in OAuth2, to try to make it more interoperable.
        . For example, Google login uses OpenID Connect (which underneath uses OAuth2). But Facebook login doesn't support OpenID Connect. It has its own flavor of OAuth2.
    + OpenAPI:
        . OpenAPI (previously known as Swagger) is the open specification for building APIs
        . FastAPI is based on OpenAPI.

    OpenAPI defines the following security schemes:
        + apiKey: an application specific key that can come from:
            . A query parameter.
            . A header.
            . A cookie.
        + http: standard HTTP authentication systems, including:
            . bearer: a header Authorization with a value of Bearer plus a token. This is inherited from OAuth2.
            . HTTP Basic authentication.
            . HTTP Digest, etc.
        + oauth2: all the OAuth2 ways to handle security (called "flows").
            . Several of these flows are appropriate for building an OAuth 2.0 authentication provider (like Google, Facebook, Twitter, GitHub, etc):
                - implicit
                - clientCredentials
                - authorizationCode
            . But there is one specific "flow" that can be perfectly used for handling authentication in the same application directly:
        + openIdConnect: has a way to define how to discover OAuth2 authentication data automatically.
            . This automatic discovery is what is defined in the OpenID Connect specification.
    
    First install python-multipart.
        E.g. pip install python-multipart.
    This is because OAuth2 uses "form data" for sending the username and password.

    Install python-jose to generate and verify the JWT tokens in Python:
        E.g. pip install "python-jose[cryptography]"

    PassLib is a great Python package to handle password hashes.
    It supports many secure hashing algorithms and utilities to work with them.
    The recommended algorithm is "Bcrypt".
        E.g. pip install "passlib[bcrypt]"

# 23. Middleware
    A "middleware" is a function that works with every request before it is processed by any specific path operation.
        + It takes each request that comes to your application.
        + It can then do something to that request or run any needed code.
        + Then it passes the request to be processed by the rest of the application (by some path operation).
        + It then takes the response generated by the application (by some path operation).
        + It can do something to that response or run any needed code.
        + Then it returns the response.

# 24. CORS (Cross-Origin Resource Sharing)
    Use CORSMiddleware
    + You can configure it in your FastAPI application using the CORSMiddleware.
        . Import CORSMiddleware.
        . Create a list of allowed origins (as strings).
        . Add it as a "middleware" to your FastAPI application.
    + You can also specify if your backend allows:
        . Credentials (Authorization headers, Cookies, etc).
        . Specific HTTP methods (POST, PUT) or all of them with the wildcard "*".
        . Specific HTTP headers or all of them with the wildcard "*".

    + The following arguments are supported:
        . allow_origins - A list of origins that should be permitted to make cross-origin requests. E.g. ['https://example.org', 'https://www.example.org']. You can use ['*'] to allow any origin.
        . allow_origin_regex - A regex string to match against origins that should be permitted to make cross-origin requests. e.g. 'https://.*\.example\.org'.
        . allow_methods - A list of HTTP methods that should be allowed for cross-origin requests. Defaults to ['GET']. You can use ['*'] to allow all standard methods.
        . allow_headers - A list of HTTP request headers that should be supported for cross-origin requests. Defaults to []. You can use ['*'] to allow all headers. The Accept, Accept-Language, Content-Language and Content-Type headers are always allowed for CORS requests.
        . allow_credentials - Indicate that cookies should be supported for cross-origin requests. Defaults to False. Also, allow_origins cannot be set to ['*'] for credentials to be allowed, origins must be specified.
        . expose_headers - Indicate any response headers that should be made accessible to the browser. Defaults to [].
        . max_age - Sets a maximum time in seconds for browsers to cache CORS responses. Defaults to 600.

# 25. SQL (Relational) Databases
    + ORMs
        . A common pattern is to use an "ORM": an "object-relational mapping" library.
        . An ORM has tools to convert ("map") between objects in code and database tables ("relations").
        . These ORMs also have tools to make the connections or relations between tables or entities.
    + File structure
    .
    └── sql_app
        ├── __init__.py
        ├── crud.py
        ├── database.py
        ├── main.py
        ├── models.py
        └── schemas.py

# 26 Bigger Applications - Multiple Files
    .
    ├── app
    │   ├── __init__.py
    │   ├── main.py
    │   ├── dependencies.py
    │   └── routers
    │   │   ├── __init__.py
    │   │   ├── items.py
    │   │   └── users.py
    │   └── internal
    │       ├── __init__.py
    │       └── admin.py

# 27. Background Tasks
    + You can define background tasks to be run after returning a response.
    + This is useful for operations that need to happen after a request, but that the client doesn't really have to be waiting for the operation to complete before receiving the response.
    Example:
        . Email notifications sent after performing an action:
        . As connecting to an email server and sending an email tends to be "slow" (several seconds), you can return the response right away and send the email notification in the background.
        . Processing data:
            For example, let's say you receive a file that must go through a slow process, you can return a response of "Accepted" (HTTP 202) and process it in the background.

# 28. Static Files
    + "Mounting" means adding a complete "independent" application in a specific path, that then takes care of handling all the sub-paths.
    + This is different from using an APIRouter as a mounted application is completely independent. The OpenAPI and docs from your main application won't include anything from the mounted application, etc.

    + The first "/static" refers to the sub-path this "sub-application" will be "mounted" on. So, any path that starts with "/static" will be handled by it.
    + The directory="static" refers to the name of the directory that contains your static files.
    + The name="static" gives it a name that can be used internally by FastAPI.