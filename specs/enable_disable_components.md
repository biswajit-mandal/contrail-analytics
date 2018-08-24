Enable/disable Contrail Analytics Components
===
# 1.      Introduction

Contrail Analytics is currently tightly copupled with kafka, users may want to use different streaming data pipelines like NSQ, NATS. So kafka should be an optional component. alarm-gen processing depends on data reading from kafka, so alarm-gen also can be optional.
```
kafka
alarm-gen
```
```snmp-collector``` and ```topology``` are needed only for underlay-overlay feature which many users may not be interested to have, so these two components also can be optional.

# 2.      Problem Statement

Installation of kafka, alarm-gen, snmp-collector and topology components in contrail analytics should be optional.

# 3.      Proposed Solution

In Contrail, a role is tied with multiple components inside it.
For example:
Role ```analytics``` implies below analytics components:
```
api
collector
query-engine
alarm-gen
snmp-collector
topology
```

Role ```analytics_database``` implies below analytics components

```
kafka
cassandra
zookeeper
```

So, to make it sync with existing approach, we need to create three additional roles to make above four components' installation optional.
```
analytics_alarm
analytics_database_kafka
analytics_snmp
```

```analytics_alarm``` role contains analytics ```alarm``` component
```analytics_database_kafka``` role contains analytics external ```kafka``` component
```analytics_snmp``` role contains analytics ```snmp-collector``` and ```topology``` components

In multi node setup, they can be installed in any node.
But, if we need to add ```analytics_alarm``` role, then ```analytics_database_kafka``` also needs to be added in any of the deploying node in the cluster.

# 3.1    Alternatives considered
None

# 3.2    API schema changes
None

# 3.3      User workflow impact
None

## 3.4      UI Changes
None


# 4 Implementation

## 4.1      Work items
### 4.1.1 Changes in contrail-ansible-deployer
Three new roles are added as discussed in [Proposed Solution](https://github.com/biswajit-mandal/contrail-analytics/blob/master/specs/enable_disable_components.md#3------proposed-solution) Section
```
analytics_alarm
analytics_database_kafka
analytics_snmp
```
As kafka can be installed irrespective of the node where analytics_database role is configured, so a new variable is added ```ANALYTICSDB_KAFKA_NODES``` which denote the kafka_nodes.

If any of the above role is not configured in a node, then we should not show the processes for that role in ```contrail-status```
To do that, we have added three new environment variables in ```/etc/contrail/common.env```.
```
ENABLE_ANALYTICS_ALARM
ENABLE_ANALYTICS_DATABASE_KAFKA
ENABLE_ANALYTICS_SNMP
```
which are used to to turn on/off the display of the contrail components status in ```contrail-status```.

### 4.1.1 Changes in contrail-container-builder
alarm-gen and collector use kafka. The kafka node details in configuration file ```contrail-collector.conf``` for collector and ```contrail-alarm-gen.conf``` for alarm-gen process are populated by ```ANALYTICSDB_KAFKA_NODES```.

This env-file ```/etc/contrail/common.env``` is passed along with contrail-status ```docker run``` arguments
```docker run --rm --name contrail-status -v $vol_opts --pid host --env-file /etc/contrail/common.env --net host --privileged ${CONTRAIL_STATUS_IMAGE}```
And internally these variables are used to show or not show the status of these analytics components.

# 5 Performance and Scaling Impact
None

## 5.1     API and control plane Performance Impact
None

## 5.2     Forwarding Plane Performance
None

# 6 Upgrade
None

# 7       Deprecations
None

# 8       Dependencies
None

# 9       Testing
## 9.1    Dev Tests
1. Do not assign the new roles, contrail-analytics should work fine and contrail-analytics should show all processes UP.
2. Assign ```analytics_database_kafka``` role in a node where ```analytics_database``` role is also assigned, contrail-analytics should work fine and contrail-status should show all processes UP.
3. Assign ```analytics_database_kafka``` role in a node where ```analytics_database``` role is not assigned, contrail-analytics should work fine and contrail-status should show all processes UP.

# 10      Documentation Impact
None

# 11      References
[1] https://nsq.io/

[2] https://nats.io/

