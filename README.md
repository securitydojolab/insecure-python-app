# Insecure Python Password Manager App

## Prequisites
* Install httpie for HTTP request via brew install httpie or apt install httpie



## Endpoint

* GET / : Initialise the application -> http GET http://localhost:8000/
* POST /create-password email=email&password=password : POST request to save email and password in the password manager -> http POST http://localhost:8000/create-password email=user@email.com password=mypassword
* GET /get-password/<email> :  Get password & email via email address -> http GET http://localhost:8000/get-password/user@email.com
* GET /redirect?url=domain : SSRF -> http GET http://localhost:8000/redirect?url=https://localhost

