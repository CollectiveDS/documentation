# Guide to use a prod fetch-repl

***

## Access prod-jump server 

We need to tunnel through a `prod-jump` connection to access the services running on prod.  To do this, edit ~/.ssh/config to alias prod-jump connection.

```
Host prod-jump
  User ec2-user
  HostName 54.210.62.101
```

This should allow for executing the following ssh command to create a successful tunnel:

```
ssh -t prod-jump ssh <prod-fetch-repl-server-ip>
```

The repo is located at `~/S71/cdshub`
The config is located at `~/config.json`

## Create a prod-fetch-repl server 

If no known prod fetch-repl is running, one can be created through the [AWS console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances).  Click the `Launch instances` dropdown in the top right and select `Launch instance from template`. Click the `Source template` dropdown and type to search for `prod-fetch-repl`.

This should populate all the needed resources.  Launch the instance.  Once created, click on the instance to go to the summary page.  In the top right, there should be a `Private IPv4 address` section.  That is the fetch-repl server ip that can be used in the ssh command:

```
ssh -t prod-jump ssh <prod-fetch-repl-server-ip>
```