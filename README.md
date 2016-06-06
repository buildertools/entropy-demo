This demo is for the [buildertools/entropy](https://github.com/buildertools/entropy) project.

# entropy-demo

Make sure that your environment is set to use your local instance and not some important endpoint.

    # get the project and start the demo
    git clone https://github.com/buildertools/entropy-demo.git
    docker-compose up -d

At this point the Entropy service and a full 3 node Swarm cluster will be running locally. This Compose environment also defines two clients for your use.

The first is a Docker client. You can use it to manipulate your local cluster. Do so now and work on the running services.

    # List the running services in the cluster
    docker-compose run --rm docker-client ps
    # Create another instance of the do-pinger service
    docker-compose scale do-pinger=1
    docker-compose run --rm docker-client ps

These services both have a "service" label.

    docker-compose run --rm docker-client ps --filter label=service
    docker-compose run --rm docker-client ps --filter label=service=PingGoogle
    docker-compose run --rm docker-client ps --filter label=service=PingDO

The other client is an Entropy serivce client. After the demo you should checkout the docker-compose.yml to see how all of this works and how to use the client directly in a real environment.

    # Start by listing the existing policies
    docker-compose run --rm entropy-client ls
    # There aren't any :(

Next create a failure policy. We are going to use the allingeek/gremlins image for the injector. The injector will have a 25% change of creating a latency spike lasting 10 seconds, every 10 seconds. For this test, target the do-pinger service.

    docker-compose run --rm entropy-client create \
        --failure latency \
        --frequency 10 \
        --probability .25 \
        --image allingeek/gremlins \
        --criteria service=PingDO

List the policies again and checkout the injectors that will have been created.

    # Get Policy list
    docker-compose run --rm entropy-client ls
    # Get Injector list
    docker-compose run --rm entropy-client lsi

These injectors are manifest as sidekicks. The sidekick containers will be named for the policy name and suffixed with their target container. These sidekicks are also richly tagged. I appologise for the copy-paste instructions, the Compose (Python) chain seems to be breaking pipes.

    # Checkout the sidekicks
    docker-compose run --rm docker-client ps
    # Grab the ID of one of the sidekick injectors and inspect it
    compose run --rm docker-client inspect --format '{{json .Config.Labels}}' <INSERT THAT CONTAINER ID>

Now that the policy has been active for some time you should be able to see latency spikes in any of the do-pinger logs for 10 seconds (or more) at a time.

    docker-compose run --rm docker-client logs -f <INSERT THE ID OF ONE OF THE do-pinger CONTAINERS>
