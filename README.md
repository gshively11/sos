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
* If integration tests fail, rollback/uninstall the deployed service.
* When master commit triggers CICD, deploy to dev/test, run tests, then deploy to staging, run tests, then deploy to prod.  This helps prevents bad services from taking down prod on a soft deployment.
* All service deployments are soft deployments.  GoLive is handled through flipr.
