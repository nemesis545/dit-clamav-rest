# DIT ClamAV REST interface

This is a python-based REST interface to ClamD inspired by https://github.com/solita/clamav-rest

### Authentication

The /scan endpoint requires basic HTTP authentication.  Authentication details are passed in via the APPLICATION_USERS environment variable which should be in the format:

    "username1::password\nusername2::password\nusername3::password"

Where `password` is a pbkdf2_sha256 hash generated by Python's passlib.  Assuming you have passlib installed, you can generate a password hash like this:

    python -c "from passlib import hash; print(hash.pbkdf2_sha256.hash('password'))"

## Running with docker-compose

    docker-compose up -d --build
    
    curl -i localhost
    
    curl -i localhost/health/definitions --user username:password


## Developing Locally

Pre-requisites

- Docker
- Python3

#### Create virtualenv and install pip dependencies

    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt

#### Start the clamd service in a docker container

    docker run -p 3310:3310 -d --name clam ukti/docker-clamav

#### Run the unit tests

    APP_CONFIG=config.LocalConfig python -m unittest tests.py

#### Run the REST api

    APP_CONFIG=config.LocalConfig python clamav_rest.py

#### Health checks

Basic Service Check

    curl -i localhost:8090


Virus definitions check
Check [Authentication](###Authentication) for `username` and `password` details


    curl -i localhost:8090/health/definitions --user username:password


returns

    200 when virus definitions are up to date
    
    401 if you didn't send credentials or credentials invalid
    
    502 when definitions are out of date
        response contains the versions
    
    500 unexpected server errors



Current clamav virus version is found using the following

```bash
host -t txt current.cvd.clamav.net

current.cvd.clamav.net descriptive text "0.101.2:58:25477:1560317340:1:63:48760:328"
```



In this example the version is `25477`

