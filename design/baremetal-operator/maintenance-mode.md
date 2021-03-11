<!--
 This work is licensed under a Creative Commons Attribution 3.0
 Unported License.

 http://creativecommons.org/licenses/by/3.0/legalcode
-->

# Add support for Maintenance Mode flag

## Status

implementable

## Summary

Currently there is no way to prevent management of BaremetalHost resources
after provisioning is completed.  The underlying Ironic API contains a
maintenance-mode flag which could be used to disable management of
BMH resources.

## Motivation

In a multi-tier deployment where one cluster deploys another, the "parent"
cluster will contain BMH resources for initial provisioning,
but the "child" cluster will later contain BMH resources that reference the
same physical hosts.

In this scenario it's necessary to prevent management operations from the
parent cluster such as asserting power state, or BMH actions on the child
cluster such as the [reboot annotation](reboot-interface.md) may fail due
to unwanted BMC interactions from the parent cluster.

This may also be useful for temporary maintenance operations
where you may wish for power management operations and polling
to be stopped for specific hosts (this is the intended use if the
underlying [Ironic API](https://docs.openstack.org/api-ref/baremetal/?expanded=set-maintenance-flag-detail#set-maintenance-flag)

### Goals

Add a maintenance flag to the BMH API which uses the underlying Ironic
interface to disable post-deploy management of a host.

### Non-Goals

TBC

## Proposal

Introduce a new BMH field `maintenance` in the spec of the
BaremetalHost object. This will be a boolean flag, which
when set to `true` will put the underlying Ironic host
into maintenance mode.

```yaml
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
metadata:
  name: worker-0
spec:
  online: true
  maintenance: true
 ...
```

TODO - we don't expose any reason here, is it enough to have
a standard reason string e.g "disabled by BaremetalHost maintenance field"?

TODO - what happens if a BMH is deleted with `maintenance: true` - do we
skip deprovisioning or should that be an independent annotation/field?

## Alternatives

It would be possible to modify the behavior of the
[pause annotation](https://docs.openstack.org/api-ref/baremetal/?expanded=set-maintenance-flag-detail#set-maintenance-flag)
such that Ironic maintenance mode is set when paused (or the host is completely
removed from Ironic), however this could pose a risk of regression to
any users expecting power-management to continue while BMO reconciliation
is paused, and it also prevents any other BMH APIs being used while
management is disabled.

The new interface could also be implemented via a new annotation, but having it in
the spec seems more consistent with some other fields which directly impact
the Ironic configuration such as the field to
[disable disk cleaning](../cluster-api-provider-metal3/allow_disabling_node_disk_cleaning.md)

## References

