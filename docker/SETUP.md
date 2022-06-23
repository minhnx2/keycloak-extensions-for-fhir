# Setup Smart on FHIR with Keycloak + Kong + HAPI FHIR

## Start HAPI FHIR server

1. Clone HAPI FHIR source code at https://github.com/minhnx2/hapi-fhir-jpaserver-starter
2. Start FHIR Server

```
cd hapi-fhir-jpaserver-starter
docker-compose up
```

## Start Keycloak + Kong

1. Start Keycloak + Kong services in this project

```
cd docker
docker-compose up
```

## Config Kong

1. Open KongA console at address http://localhost:1337
2. Create new admin account if it's not existed
3. Connect to Kong Admin via Dashboard screen or navigate to Connections menu using following connection info:

```
Connection Type: default
Name: local
Kong Admin URL: http://host.docker.internal:8001
```

3. Import sample snapshot config from "./kong/snapshots/" folder in this project via Snapshots menu in KongA console

a. Click Import From File menu in Snapshots screen
b. Choose ./kong/snapshots/kong-oidc-introspect-keycloak.json to import
c. Click Details button on newly created record
d. Click Restore buttin
e. Select all object types to import

Note: Re-generate client secret in Keycloak console and replace value in Kong OIDC connection info if required

## Config Keycloak

1. Open Keycloak console at address http://localhost:8180
2. Navigate to Add Realm screen
3. Import Keycloak config from "./docker/keycloak/fhir-realm.json" folder
4. Create new user via Users menu
5. Open newly created user, navigate to Attributes tab
6. Assign Patient Ids to current user via resourceId key

Ex:
```
resourceId=2

Or

resourceId=2 3
```

# Access to services via Kong proxy

1. Add this address to OS host file (/etc/hosts on Mac/Linux, <WINDOWS_DRIVER_NAME>:\Windows\System32\Drivers\etc\hosts on Windows)

```
127.0.0.1 host.docker.internal
```

2. Access sample Kong Proxy route by using below url

1. Test without bearer token

```
curl --location --request GET 'http://localhost:8000/api/fhir/CareTeam/1'
```

2. Test with bearer token

a. Authenticate and get token from Keycloak with following config, note that Auth domain path must be the same between Kong and client app:

-   Grant type: Authorization Code
-   Callback URL: <any>
-   Auth URL: http://host.docker.internal:8180/auth/realms/fhir/protocol/openid-connect/auth
-   Access Token URL: http://host.docker.internal:8180/auth/realms/fhir/protocol/openid-connect/token
-   Client ID: fhir-client
-   Client Secret: <client secret>
-   Scope: openid launch/patient
-   Client Authentication: Send as Basic Auth header

b. Use above generated bearer token to call APIs, replace <Token Value> to the generated JWT token value in previous step

```
curl --location --request GET 'http://localhost:8000/api/fhir/CareTeam/1' \
--header 'Authorization: Bearer <Token Value>'
```
