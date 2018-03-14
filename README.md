This repository contains a SQL Server data tools project and a Jenkinsfile written using the opinionated declarative syntax for a build pipeline that:

- Checks the project out from SCM
- Spins up a container to deploy the DacPac to
- Deploys the DacPac to the container
- Tears down the container
