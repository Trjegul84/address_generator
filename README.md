# wallet

Generating and displaying cryptocurrency addresses

# Overview

This is a Django application for generating and displaying cryptocurrency addresses via REST API.

The addresses are generated by using hierarchical deterministic key derivation with a single master private key.
The derivation path is the input parameter of the address generator.

Currently supported crypto currencies:
- BTC bitcoin
- ETH ethereum
- LTC litecoin
- TRX tron


# Bootstrap

Install Python 3.6 or newer.

Create a python virtual environment:
```
python -m venv .venv
```

Activate the virtual environment:
```
. .venv/bin/activate
```

Install [Python library dependencies](requirements.txt):
```
pip install -r requirements.txt
```

Before running the service, set the environment variables:
```
export ZEPLY_XPRIVATE_KEY='xprv9s21ZrQH143K24t96gCaezzt1QQmnqiEGm8m6TP8yb8e3TmGfkCgcLEVsskufMW9R4KH27pD1kyyEfJkYz1eiPwjhFzB4gtabH3PzMSmXSM'
```
This is a master / root private key for generating a private key for certain address. In this case ZEPLY_XPRIVATE_KEY was taken from HD wallet examples but it can be generated with a library function or algorithm.


In case of `ValueError: unsupported hash type ripemd160`,
apply the [following](https://stackoverflow.com/questions/69922525/python-3-9-8-hashlib-and-ripemd160).


# Usage

Start the application:
```
python manage.py runserver
```

The application can be run in a docker container:
```
docker run -it --rm --name address_generator -v "$PWD":/home/app -p 8000:8000 -w /home/app python:3.8 bash
```

The application accepts the following HTTP requests:

- Create a BTC address:
```http
POST /addresses/ HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
    "currency": "BTC",
    "path": "m/44/0"
}
```
Response:
```http
HTTP/1.1 201 OK
Content-Type: application/json
{
    "id": 1,
    "currency": "BTC",
    "address": "1EuXukqwKVKxPrv8xfUsu5Jq5anmM4XWkp"
}
```

- List addresses:
```http
GET /addresses/ HTTP/1.1
Host: localhost:8000
```
Response:
```http
HTTP/1.1 200 OK
Content-Type: application/json
[
    {
        "id": 1,
        "currency": "BTC",
        "address": "1EuXukqwKVKxPrv8xfUsu5Jq5anmM4XWkp"
    },
    {
        "id": 2,
        "currency": "ETH",
        "address": "0xEb054b3e35a62e949eC0B41411375d19426338bA"
    }
]
```

- Retrieve an address with a given ID `1`:
```http
GET /addresses/1/ HTTP/1.1
Host: localhost:8000
```
Response:
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 1,
    "currency": "BTC",
    "address": "1EuXukqwKVKxPrv8xfUsu5Jq5anmM4XWkp"
}
```


# Test

To run all tests, run from the root directory of the repository:
```
python manage.py test
```

A [unit test](addresses/tests.py) tests the address generator module with the `generate_address` API.
In the unit test, all the dependencies (usually the libraries imported in the module under test) are patched / mocked.
Only the functionalities of the module under test have to be tested, but not the dependencies,
e.g. that `hdwallet` can generate an address must not be tested.

A [REST API end-to-end test](tests.py) tests that each endpoint can be requested and the response is proper.


# Security

The paths and `ZEPLY_XPRIVATE_KEY` master / root private key are the sensitive data.
Although the private key is always needed to generate an address, the paths are never returned in any response.

To increase security,
[django can be used with SSL](https://timonweb.com/django/https-django-development-server-ssl-certificate/).

For using SSL,
[a certificate needs to be generated](https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl):
```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes -subj '/CN=localhost'
```

The `ZEPLY_XPRIVATE_KEY` environment variable has to be provided by a secret manager like
[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/).