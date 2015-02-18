# Two demos:

 1. Run a fig config through swarm and put the config on its own network

 2. Move a container with its IP from one host to another (optional:
 when it fails).

## Run a fig config through swarm

Currently if you do this you get an app distributed among hosts that
can't talk to itself. (Is that accurate?)

Running weave on each host, we can assign each app its own subnet and
give the containers in it IPs in the subnet; and, each container
should be able to resolve the other containers by name.

### Implementation

When a fig config is played through the swarm master, it makes a new
name for the application, takes an unallocated subnet and assigns it
to the name. (Problem: how does swarm know all the containers are from
the same application? For now it might have to be an explicit env
entry that it looks for)

Each container in the application gets a hostname of
`<fig-name>.<app-name>.weave.local` and an IP address from the subnet.

The IP address is 'attached' to the container by swarm giving it an
extra environment entry. Weave notices this and gives the container an
interface on the weave network with the IP address, and updates
weaveDNS with the hostname. (Are there better ways?)

To see when it needs to act, weave runs as a proxy to the docker
daemon, on each host.

## Move a container with its IP from one host to another

With the configuration above (proxies on each host, some bookkeeping
by swarm) enacting this should be as simple as starting the new
container on the target host, with the IP address in the environment
entry and hostname, and stopping the old container.

How is it triggered though? It could simply be a script that stops and
starts the container, with swarm doing the right thing by virtue of it
knowing about the application. (Problem: why would swarm start it on a
different host? Could it be "hinted" with an argument to docker run,
or otherwise?)

## Unresolved issues and hacks

Most of it is a hack!

 * Proxying the docker daemon is not great; that's where we'd want
   network extensions to work for us
 * Likewise, using env entries to communicate IP addresses is not
   ideal
 * Where should subnet allocation really take place
 * Where should IP allocation really take place
 * Publishing ports, how does that work with swarm
