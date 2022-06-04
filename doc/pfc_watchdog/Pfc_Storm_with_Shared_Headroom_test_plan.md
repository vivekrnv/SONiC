# PFC Storm with Shared Headroom test plan

## Motivation

When the Buffer occupany is filled upto Shared Headroom and a PFC Storm is seen at the same time, PFC WatchDog on the Melanox Devices applies Zero Buffer Profiles on Ingress PG and Egress Queues. 

When the PFC_WD is restored and the original buffer profile is applied back the ASIC has a bug which thinks that the occupancy is still above Xon and keeps sending PFC frames to the peer link even though the original occupancy is under Xon.

Owing to this, the community decided not to apply ZeroBufferProfile to Ingress PG during a PFC Storm.

Thus the test case is added to cover this scenario

**Note:** 
+ This test case is only intended for Mellanox Platforms
+ This test case requires an RPC image 

## Overview
+ Verify if the shared headroom is enabled
+ Simulate a buffer congestion in the ingress PG for the DUT source port so that the occupancy crosses into the shared headroom region
   - Achieve this by closing the dut tx port `sai_thrift_port_tx_disable` call https://github.com/Azure/sonic-mgmt/blob/master/tests/saitests/switch.py#L624.
+  - Send pkts 
+ Trigger a PFC Storm directed towards the DUT source port
+ PFC Watchdog is triggered and Zero buffer profile is applied only on egress
+ Drain the Ingress buffers to drop the occupancy under Xon
+ Check the Tx PFC Counters on the DUT source ports and verify nothing is sent after the ingress buffers are drained






