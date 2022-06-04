# PFC Storm with Shared Headroom Test Plan

## Motivation

When a PFC Storm is seen, PFC WatchDog on the Melanox Devices applies Zero Buffer Profiles on Ingress PG and Egress Queues. However if the buffer occupany has already crossed Shared Headroom (i.e. Shared Buffer + Xon) when a PFC Storm is detected, a bug was recently found on mellanox platform. 

The bug is seen after the PFC Watch dog is restored. After PFC_WD is restored, the original buffer profile is applied back. The ASIC had an accounting bug which makes it think that the occupancy is still above Xoff and the DUT keeps sending PFC frames to the peer link even though the actual occupancy is under Xon.

Although this was fixed in the FW, the community decided not to adopt this solution because of an extra delay incurred in this Solution. Thus the community decided to modify the SONiC Flow to not apply ZeroBufferProfile to Ingress PG during a PFC Storm.  

Thus the test case is added to cover this scenario and check if the faulty accounting scenario is not seen. 

**Note:** 
+ This test case is only intended for Mellanox Platforms
+ This test case requires an RPC image

## Test Plan
+ Verify if the shared headroom is enabled
+ Make sure buffer occupancy crosses into the shared headroom region
   - Achieve buffer congestion by closing the dut tx port using `sai_thrift_port_tx_disable` API: https://github.com/Azure/sonic-mgmt/blob/master/tests/saitests/switch.py#L624.
   - Send pkts from the PTF docker which are destined to egress out of the dut tx port.
   - Make sure to send atleast num_pkts_pfs_frame + private_headroom pkts pkts
   - num_pkts_pfs_frame: num of pkts required to be sent in order to trigger a PFC frame from the DUT. More on this here: https://github.com/Azure/sonic-mgmt/blob/master/tests/qos/files/mellanox/qos_param_generator.py
   - private_headroom_pkts is specific to mellanox which is in the order of a few pkts.
    
+ Trigger a PFC storm directed towards the DUT Rx port
+ PFC Watchdog is triggered
+ After PFC WD is restored, drain the Ingress buffers to drop the occupancy under Xon
  - Achieve this by re-opening dut tx port using `sai_thrift_port_tx_enable` API.
  - This'll drain the dut rx buffers and the occupancy falls below Xon.
+ Check the Rx PFC Counters on the DUT after the packets are drained. They shouldn't be incremented as the occupancy has fallen below Xon
