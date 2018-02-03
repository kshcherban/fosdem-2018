# Contents
* [Saturday](saturday)
    1. [Monitoring legacy Java applications with Prometheus](#monitoring-legacy-java-applications-with-prometheus)
    1. [PostgreSQL replication in 2018](#postgresql-replication-in-2018)

# Saturday

## Monitoring legacy Java applications with Prometheus
### Logfiles monitoring
logs parsing
tool: https://github.com/google/mtail
grok\_exporter turns logs into metrics
prometheus labels to parse logs
turns logs into metrics

### Blackbox exporter

blackbox exporter calls url and parses response with modules, like http 200
should cover all possible endpoints

### JMX
java visualvm is a JDK tool to connect to JMX
prometheus has parser for jmx xml shit
jmx exporter
polling jmx data may cause significant performance penalty
### DIY java agent

Hooks into classes and collects metrics
promagent.jar java exporter


## PostgreSQL Replication in 2018
Back then postgres had no replication! Not a case now.
Both external and internal solutions for replication.
Replication can be done at different layers:
    - application
    - app in database
    - trigger based
    - database logical
    - database physical
    - OS - DRBD
    - Hardware based replication - SAN

### SAN replication
Block level, transparent, magic inside.
Fails hard.

### Operating system
DRBD the most common, implemented as OS driver, slow.

### Database physical
WAL based, file based.
Synchronous mode (replication):
    - standby nodes in readonly mode
    -  three modes:
        - simple
        - firsthttps://repmgr.org/
        - quorum
**Streaming replication**: very easy to set up, hard to get wrong, it's working,
built in, use a user with `REPLICATION` privilege, `-X stream` with postgres 10 is default, different OSes and platforms are not supported,
replicates everything, can replicate in one direction,
no cluster management, doesn't support (easy) fallback: might be uncommitted data.

**Patroni** tool for replication and failover and cluster management.
https://github.com/zalando/patroni

**repmgr** cluster replication manager (https://repmgr.org/)

**PAF** pacemaker based, manages virtual ip

### Database logical
wal\_level='logical'  
Reconstruct changes on a raw basis. Table level or other partial replication. Might work most of the times but not **all** the time. Foreign keys may be an issue.

Drawbacks:
  - no schema replication
  - doesn't replicate sequences
  - not good for HA replication
  - not complete

Extension: **pglogical** has more capabilities today, was partly integrated in Postgres 10. Does support sequence replication, column and raw based filtering.

### App in database
Easy to break, not easy to work with. Slony, Bucardo, Longdiste.
Multimaster is tricky. Completely transparent multimaster doesn't exist.
**BDR** a fork of Postgres that provides multimaster.

Application controlled replication makes app very complex.


### Summary
For HA use streaming replication, mix sync and async replication,
consider patroni or repmgr.
Don't use logical replication for HA!

Read query offloading is for query replication.

Data distribution is for logical one.

QA:
- How much latency is added for sync replication?
Depends on hardware and network.

- Can you create separate indexes on read replicas?
With binary replication NO.

- Pgpool for replication?
Don't use it. May be used for balancing.

