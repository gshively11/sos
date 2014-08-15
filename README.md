sos
===

Service-oriented Stack - Technologies and strategies for making a service-oriented application.

# Building Blocks

* Host OS: REHL 6 or 7
* Language/Runtime: javascript/node
* Web/Proxy Server: nginx
* REST Framework: restify or express (more research needed)
* REST Documentation: swagger
* Task Runner: gulp.js
* Test Framework: mocha, chai, sinon, proxyquire
* Flow Control: async.js
* Client-side Module Definition: browserify
* Continuous Integration Server: bamboo
* Search: elasticsearch
* Log Forwarding: logstash
* Dashboards: kibana
* Logs and Metrics: ELK + redis (backpressure)
* Self-service Operations: rundeck
* Application Configuraiton and Feature Flipping: node-flipr-etcd
* SSO: node-gd-auth
* Proxy between node processes: node-http-proxy
* Scaffolding: yeoman
* Message Bus: ??? - kafka or RabbitMQ
* Server Configuration: ??? - chef, puppet, ansible
* Log Writer: ???
* Service Discovery: ???

# Feature Request to Go Live
1. Create branch, make changes
2. When changes can be deployed, create PR (doesn't have to be feature complete)
3. PR automatically triggers CICD: build, test, deploy to dev/test
4. Code is peer reviewed
5. Merge to master
6. Merge to master automatically triggers CICD: build, test, deploy to dev/test/staging/prod
7. Update service hash in flipr config, commit

# More on CICD
* All service deployments are soft deployments.  GoLive is handled through flipr.
* If integration tests fail, rollback/uninstall the deployed service.
* When master commit triggers CICD, deploy to dev/test, run tests, then deploy to staging, run tests, then deploy to prod.  This helps prevents bad services from taking down prod on a soft deployment.

# Example API Request
Let's say I want to call our API to get information on a product (GET /v1/products/1234).  How does that request get from my browser all the way to the service that will get me the informatin I need?

1. HTTP Request GET /v1/products/1234
2. DNS is pointing to a load balancer, which round-robin routes to a web server.
3. Nginx, listening on 80/443 will accept request, terminate SSL, and route to API port based on the host header
4. Main API will perform authentication and set user context.
5. Based on the route, API determines that we need to talk to products-service
6. API will check flipr config to see which version of products-service it should talk to
7. API will then check with service discovery to see where that version products-service lives
8. API will then proxy the request to the appropriate instance of products-service using node-http-proxy
9. Products-service will accept the request and perform any authorization needed based on the user context
10. Products-service will get the product information and send the response
11. API will receive the response and forward it on to nginx
12. Nginx will receive the response and forward it on to the load balancer
13. Load balancer will receive the response and send it back to the client

# Strategies for dealing with an explosion of services
At the nginx level, we should be generating a uuid for all requests, which is added as a header.  This header will be added to the nginx access logs.  It will also be persisted during all internal proxying of the request, and all services will use the uuid when writing logs.  This will allow us to trace all the steps a request takes in our system.  Furthermore, if we send an error response back to the client, the UI should display the uuid to the user, telling them to reference this uuid when reporting errors.  This will allow us to immediately start diagnosing a specific client's problem without having to dig for it based on timestamp and/or user matching.

Here's an nginx module to add the request id: https://github.com/newobj/nginx-x-rid-header

We need an automated process to clean up old services on our servers.  Because each deployment spawns a separate process, we will eventually eat up all our resources (ports, memory) if we don't clean them up.  Ideally, the automated process could be told something like "remove any service that hasn't received any traffic after X number of days".  If completely automated removal is determined to be too risky, we need to provide a tool that can easily give us the information we need to make the kill decision and a button click to perform it.
