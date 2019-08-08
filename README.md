# Running Kong in docker

create network

    docker network create kong-net

run postgres

    docker ru -d --name kong-database \
            --network=kong-net \
            -p 5432:5432 \
            -e "POSTGRES_USER=kong" \
            -e "POSTGRES_DB=kong" \
            postgres:9.6

prep postgres

    docker run --rm \
         --network=kong-net \
         -e "KONG_DATABASE=postgres" \
         -e "KONG_PG_HOST=kong-database" \
         kong:latest kong migrations bootstrap

run kong

    docker run -d --name kong \
         --network=kong-net \
         -e "KONG_DATABASE=postgres" \
         -e "KONG_PG_HOST=kong-database" \
         -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
         -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
         -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
         -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
         -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
         -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
         -p 8000:8000 \ ##listen to http
         -p 8443:8443 \ ##listen to https
         -p 8001:8001 \ ##admin api
         -p 8444:8444 \ ##admin api over https
         kong:latest

test
    
    curl http://localhost:8001      

# set up ecs via cloudformation using docker

## create dockerfile with aws cli and curl
    docker build -t aws-ecs -f Dockerfile.aws-ecs .

## create cloudformation template to spin up ecs


## add service to kong api-gateway
ref: https://docs.konghq.com/1.2.x/getting-started/configuring-a-service/


## to-do's
1. set up tls 
