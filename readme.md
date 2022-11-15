# Deta.sh HTTP API 

Reference: https://raw.githubusercontent.com/deta/docs/master/docs/cli/commands.md

---

## Prerequisites 

### Sign Up & Create Access Token

- Create an account here https://web.deta.sh/

- Login & click Create First Project & select default region

- Click Settings -> Create Token

- Save this token somewhere safe, like a password manager

Moving forward it is assumed you will have some environment variables set in your terminal session.

The code snippets will be sourcing `./.env`, so if you are following along, that would be the place to save these environment variables!

``` 
DETA_ACCESS_TOKEN=XXXXXXXX_YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
echo "DETA_ACCESS_TOKEN=${DETA_ACCESS_TOKEN}" >> ./.env
echo "DETA_ACCESS_KEYID="$(echo -en ${DETA_ACCESS_TOKEN} | awk -F'_' '{print $1}')""  >> ./.env
echo "DETA_ACCESS_KEYSECRET="$(echo -en ${DETA_ACCESS_TOKEN} | awk -F'_' '{print $2}')""  >> ./.env
```

---

## HTTP API Access

### Building The Signature Header & Getting Your spaceID

To craft our signature we need to gather / decide on the following info
 - `HTTPMethod='GET'`
 - `URI='/spaces/'`
 - `Timestamp="$(date +%s)"`
 - `ContentType=''`
 - `RawBody=''`

To generate our signature string we can do the following:

```
echo -en "$HTTPMethod\n$URL\n$Timestamp\n$ContentType\n$RawBody\n" | openssl dgst -sha256 -hmac $DETA_ACCESS_KEYSECRET | awk '{print $NF}
```

If you were to resolve those variables, it would look like this:

```
echo -en "GET\n/spaces/\n1234567890\n\n\n" | openssl dgst -sha256 -hmac "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY" | awk '{print $NF}'
```

So lets build our curl command:

```
source .env
HTTPMethod='GET'
URI='/spaces/'
Timestamp="$(date +%s)"
ContentType=''
RawBody=''
SIGKEY="$(echo -en "$HTTPMethod\n$URI\n$Timestamp\n$ContentType\n$RawBody\n" | \
          openssl dgst -sha256 -hmac $DETA_ACCESS_KEYSECRET | \
          awk '{print $NF}')"
SIG="v0=${DETA_ACCESS_KEYID}:${SIGKEY}"
RESP="$(curl -H "User-Agent: Go-http-client/1.1" \
             -H "X-Deta-Signature: $SIG" \
             -H "X-Deta-Timestamp: $Timestamp" \
             -H "Accept-Encoding: gzip" \
             -H "Accept:" \
             https://v1.deta.sh${URI})"
spaceID=$(echo -ne $RESP | python3 -c 'import sys; import json; print(json.loads(sys.stdin.read())[0]["spaceID"], end="")')
echo $spaceID
```

from this point on we will assume you have saved your spaceID number as an environment variable `spaceID` inside `./.env`

```
echo "spaceID=${spaceID}" >> .env
```

### Listing Your Projects

```
source .env
HTTPMethod='GET'
URI="/spaces/$spaceID/projects"
Timestamp="$(date +%s)"
ContentType=''
RawBody=''
SIGKEY="$(echo -en "$HTTPMethod\n$URI\n$Timestamp\n$ContentType\n$RawBody\n" | \
          openssl dgst -sha256 -hmac $DETA_ACCESS_KEYSECRET | \
          awk '{print $NF}')"
SIG="v0=${DETA_ACCESS_KEYID}:${SIGKEY}"
RESP="$(curl -H "User-Agent: Go-http-client/1.1" \
             -H "X-Deta-Signature: $SIG" \
             -H "X-Deta-Timestamp: $Timestamp" \
             -H "Accept-Encoding: gzip" \
             -H "Accept:" \
             https://v1.deta.sh${URI})"
echo $RESP
```

