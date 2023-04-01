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
* Service expect to find a configuration file at `.config/<env>.json` with configuration
values. Please see `.config/config.js` for more details and alternative
environment variables.
* Service was tested with node 14.2


## Runing:
* Run `npm install`
* Run `npm start`
