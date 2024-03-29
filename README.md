# Kong Testing

## Run

docker network create kong-net

docker run -d --name kong-database \
   --network=kong-net \
   -p 5432:5432 \
   -e "POSTGRES_USER=kong" \
   -e "POSTGRES_DB=kong" \
   postgres:9.6

docker run --rm \
   --network=kong-net \
   -e "KONG_DATABASE=postgres" \
   -e "KONG_PG_HOST=kong-database" \
   kong:latest kong migrations bootstrap


docker run -d --name kong \
  --network=kong-net \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  kong:latest
  
## Add Service - Yoisho Currency
 
 curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=currency3' \
  --data 'url=http://backend.yoisho.dob.jp/fx'
  
## Add Route

curl -i -X POST \
--url http://localhost:8001/services/currency3/routes \
--data 'hosts[]=localhost' \
--data 'paths[]=/fx' \
--data 'strip_path=true' \
--data 'methods[]=GET'

## Consume Route

curl http://localhost:8000/fx/currency?currency=USD

## API Key Auth

### Activate Plugin on Service
curl -X POST http://localhost:8001/services/currency3/plugins --data "name=key-auth"

### Add Consumer & Generate Key

curl -X POST http://localhost:8001/consumers --data "username=user1"   
curl -X POST http://localhost:8001/consumers/user1/key-auth --data ""

### Consume API with Key

curl --header 'apikey: rVmNSTvgjGuvhax8nr1yc8XpRPZ3LvTW' localhost:8000/fx/currency?currency=USD

## Basic Auth

### Activate Plugin on Service

curl -X POST http://localhost:8001/services/currency4/plugins \
  --data "name=basic-auth"  \
  --data "config.hide_credentials=false"
  
### Add Consumer & Set Credentials

curl -X POST http://localhost:8001/consumers --data "username=user1"   
curl -X POST http://localhost:8001/consumers/consumer1/basic-auth \
    --data "username=me" \
    --data "password=bla"
    
### Consume with Basic Auth

curl -u me:bla localhost:8000/fx2/currency?currency=USD

## URLs

* http://localhost:8001/services
* http://localhost:8001/routes
* http://localhost:8001/consumers
* http://localhost:8001/plugins


