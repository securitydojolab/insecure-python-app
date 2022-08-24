# Insecure Python Password Manager App

## Endpoint

* GET / : Initialise the application
* POST /create-password email=email&password=password : POST request to save email and password in the password manager
* GET /get-password/<email> :  Get password & email via email address
* GET /redirect?url=domain : SSRF
