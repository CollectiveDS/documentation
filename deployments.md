# Guide to explain deployments

Deployments are handled by Jenkins, which is accessible through Tailscale at http://jenkins:8080

If the URL is unreachable, ensure that Tailscale is enabled and the `jenkins` device shows up as a tagged device:

![Tailscale Jenkins](/images/deployments_tailscale.png?raw=true)

If the tagged device is not present, you will need to request Ben adds a permission to your Tailscale account



## BE Deployments

![Jenkins BE Deployments Image](/images/deployments_jenkins_dashboard.png?raw=true)

### BE-CDSHUB

This job handles building CDSHUB for prod and stage.  This is done through build parameters:


![BE-CDSHUB Build Parameters](/images/deployments_be_cdshub.png?raw=true)
![BE-CDSHUB Build Parameters](/images/deployments_be_cdshub_checkboxes.png?raw=true)

- The `HASH` parameter is the github hash for the branch you wish to deploy.  Github hashes can be found multiple ways through the GUI, or the `git rev-parse HEAD` Git CLI command when on the desired branch.
- The `HOSTS` parameter tells what exactly to deploy.  For new CDSHUB Clojure code changes, usually `api` is sufficient
- The `TIMBRE` parameter tells what level of logging to report for fetch servers to and is reflected in Papertrail.  Shouldn't ever need to be changed
- The `API_TIMBRE` parameter is essentially the same, but for the API server.  Shouldn't ever need to be changed
- The `TARGET` parameter is important to decide what is being built.  Selecting `PROD` will deploy and become our live prod instance, and `STAGE` will deploy to our stage environment
- The `REBUILD` checkbox when unchecked will look in s3 for a jar file and copy that already built jar file and deploy that.  Will not recompile any code as it will just run what was compiled the last time we deployed that hash
- The `REBUILD_DEPS` checkbox will delete the `.m2` directory and pull all new dependencies
- The `RUN_ANSIBLE` checkbox kicks off the Ansible servers prior to deploying
- The other `SQL` fields are for adding new tables/columns on deployment to ensure BE tickets with table modifications will not cause table errors when deployed

## How to rollback prod

The basic workflow for rolling back prod should be finding the last known build that worked for your desired host (e.g. `api` or `all` if it's something to do with Clojure API issues) by using Jenkins and going through the most recent builds and selecting `Previous Build`

![Jenkins BE Last Known Deployment](/images/deployments_be_cdshub_rollback_example.png?raw=true)

Copying the `HASH` field and kicking off a deployment with the target `PROD` should be all you need as it will find the jar file in s3 and complete the deployment

## How to update prod config

If a server URL changes and needs to be updated in `config.json`, make the changes in the https://github.com/CollectiveDS/devops-config repo within the `prod-api` and `prod-fetch` areas.  The `master` branch will get sync'd to s3 and become the new `config.json` file when an `ansible` host is deployed
