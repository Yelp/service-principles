# How GrandRounds defines "Production Ready"

To launch a service in production your service must have the following:
- [ ] Automated configuration and deployment
- [ ] Security review and access controls in place
- [ ] Monitoring configured and verified. Alerting setup
- [ ] Scaling plan and Availablity plan defined
- [ ] Backup and Recovery plans exist and have been tested

## Automated configuration and deployment
Service/system should be defined through configuration files, base images, setup scripts. There should be no manual configuration or setup of system. Instructions should be available along with any setup scripts. Instructions should be detailed enough to hand over to someone else to run. The system should be able to be created in the common environments we use (production, UAT, integration, CI, development) using automated tooling. The automation scripts serve as 

## Security review and access controls
Submit your review of the security of the service. The review should include what attack surfaces are created by this service and how you mitigate them. Detail who/what systems access this new service and how. Define the service’s ACLs and for the deployment of the system. Describe the types of security and encryption used. Include details of the PHI/PII types. Re-encryption plan/schedule is needed as well.

## Monitoring/Alerting configured and verified
Set up your monitoring system for the service in each environment. This should include status health checks for the ELB (if applicable), system level metrics, service logs, reporting metrics and alerting thresholds.

## Scaling plan and Availability plan
Describe the load characteristics of the system. Prove these characteristics by including load testing as part of your automated tests. 10x the anticipated usage of the service and describe how and what will need to be changed to support this new load. Our current default configuration for availability is to have two duplicate services running. Your service must be configured to have two (or more) hosts in all environments. In your availability plan discuss deployments and how your service will handle multiple versions running at the same time.

## Backup and recovery plan
Describe what needs to be backed up and how it will get backed up. Include details on retention and what is expected. For the recovery plan for the system, describe if the database/web service are lost how it will be restored, what are the potential losses, what is the expected recovery time. Run through a recovery scenario and provide the details on the issues encountered and how they will be resolved. An Infra team member will follow your recovery plan to verify it is repeatable.


