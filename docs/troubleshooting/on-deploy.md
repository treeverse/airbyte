---
description: Common issues and their workarounds related when trying launch Airbyte
---

# On deployment

## Stuck in onboarding, can’t skip or do anything
To full reset Airbyte, you need to also delete the docker volumes associated with Airbyte. This is where data is stored.
Assuming that you are running Airbyte by running `docker-compose up`, then what you need to do is:
* Turn off Airbyte completely: `docker-compose down -v`
* Turn Airbyte back on: `docker-compose up`

that should handle you getting reset to the beginning.
I would be curious if we can see the logs associated with the failure you are seeing. I would say if after you reset you run into it again we can debug that.


### I have run `docker-compose up` and can not access the interface

- If you see a blank screen and not a loading icon:
  
Check your web browser version; Some old versions of web browsers doesn't support our current Front-end stack.

- If you see a loading icon or the message `Cannot reach the server` persist:

Check if all Airbyte containers are running, executing: `docker ps`

```text
CONTAINER ID   IMAGE                            COMMAND                  CREATED        STATUS        PORTS                              NAMES
f45f3cfe1e16   airbyte/scheduler:1.11.1-alpha   "/bin/bash -c './wai…"   2 hours ago    Up 2 hours                                      airbyte-scheduler
f02fc709b130   airbyte/server:1.11.1-alpha      "/bin/bash -c './wai…"   2 hours ago    Up 2 hours   8000/tcp, [...] :::8001->8001/tcp  airbyte-server
153b2b322870   airbyte/webapp:1.11.1-alpha      "/docker-entrypoint.…"   2 hours ago    Up 2 hours   :::8000->80/tcp                    airbyte-webapp
b88d94652268   airbyte/db:1.11.1-alpha          "docker-entrypoint.s…"   2 hours ago    Up 2 hours   5432/tcp                           airbyte-db
0573681a10e0   temporalio/auto-setup:1.7.0      "/entrypoint.sh /bin…"   2 hours ago    Up 2 hours   6933-6935/tcp, [...]               airbyte-temporal
```
You must see 5 containers running. If you are not seeing execute the following steps:
* `docker-compose down -v`
* `docker-compose up`
Keep in mind the commands above will delete ALL containers, volumes and data created by Airbyte.
  We do not recommend this is you already deploy and have connection created.

First, let's check the server logs by running `docker logs airbyte-server | grep ERROR`. <br>
If this command returns any output, please run `docker logs airbyte-server > airbyte-server.log`. <br>
This command will create a file in the current directory. We advise you to send a message on our #issues on Slack channel

If you don't have any server errors let's check the scheduler, `docker logs airbyte-scheduler | grep ERROR`. <br>
If this command returns any output, please run `docker logs airbyte-scheduler > airbyte-scheduler.log`. <br>
This command will create a file in the current directory. We advise you to send a message on our #issues on Slack channel

If there is no error printed in both cases, we recommend running: `docker restart airbyte-server airbyte-scheduler` <br>
Wait a few moments and try to access the interface again.

### `docker.errors.DockerException`: Error while fetching server API version

If you see the following error:

```text
docker.errors.DockerException: Error while fetching server API
version: ('Connection aborted.', FileNotFoundError(2, 'No such file or
directory'))
```

It usually means that Docker isn't running on your machine \(and a running Docker daemon is required to run Airbyte\). An easy way to verify this is to run `docker ps`, which will show `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?` if the Docker daemon is not running on your machine.

This happens (sometimes) on Windows system when you first install `docker`. You need to restart your machine.


### Getting a weird error related to setting up the Airbyte server when running Docker Compose -- wondering if this is because I played around with Airbyte in a past version?

If you are okay with losing your previous Airbyte configurations, you can run `docker-compose down -v` and that should fix things then `docker-compose up`.
