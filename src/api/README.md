# Ping API micro-service

This service counts pings

## Endpoints:
* `/`
* `/ping` (POST)
* `/pinger`
* `/isAlive`
* `/probe/liveness`
* `/probe/readiness`


## Configuration:
* Service expect to find a configuration file at `.config/confg.js`
* * redis_host
* * redis_port
* * redis_pass
* Alternatively, an environment variable `REDIS_URL` can be provided.
   
