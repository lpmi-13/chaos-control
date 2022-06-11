# Chaos Control (Dockerized)

This is a proof of concept for a controlled chaos experiment inside of a gitpod environment (or locally, if you prefer).

## The stack

- A django app that allows users to create polls and vote on them (Do we want to scale this horizontally? We probably can, and it would be more authentic that way)

- Traefik running as a reverse proxy/load balancer. This will give us latency, traffic and error rate for grafana

- A postgres database that stores the poll configuration and state

- ELK (Filebeat grabs the container logs from the the host, sends them on to elasticsearch, and those are indexed and displayed via kibana)

- Grafana (TBD: We definitely want user-facing metrics like the latency, traffic, error rate, and _maybe_ some sort of measure of saturation, which maybe we can use container CPU/RAM for...)

- Prometheus (We can probably instrument the user journeys through the app...create poll, answer poll, etc)

## The scenarios

We're going to set up three distinct failure modes, and trigger them probabilistically. As in, there will be three possible things that will go wrong, but the we won't know which one until we fire up the scenario.

- Network partition

To simulate something like network flapping, we can use the `tc` utility to drop a certain percentage of packets between the load balancer and the app server, or we could even completely stop traffic to one of them. We can also stop all connections to the database to simulate a firewall update that cuts off access.

- Disk full

To simulate one or more of the app servers filling their disk with logs, we can just write a whole bunch of junk files to the containers. I'm not exactly sure how easily we can use this to put the containers in a crash loop, but we'll see what we can do.

- Some sort of resource exhaustion (CPU or RAM)

We could just reset the limits for the containers to be super tiny, and then force a cycling, meaning they wouldn't have enough resources to start or service a high number of requests. I don't immediately know what the mechanism for this would be, but we can work something out.


## Events

- We'll have a process to trigger one of the above scenarios, maybe using the chaos toolkit, or something custom.

- We'll have some options to hook up alerts (possibly based on grafana, that can send directly to slack...dunno if discord is an option), and you'll receive an alert about something that went wrong.

- You dive into the dashboards and logs and see if you can figure out which of the 3 scenarios it is.

- After you've identified the issue, fix it, and then see if you can validate the fix.
