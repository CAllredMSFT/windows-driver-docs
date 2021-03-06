---
title: Miniport adapter OID request serialization
description: Miniport adapter OID request serialization
keywords:
- OIDs WDK networking , miniport adapter requests
- miniport adapters WDK networking , OID requests
- adapters WDK networking , OID requests
- object identifiers WDK networking
- OID serialization
ms.date: 04/20/2017
ms.localizationpriority: medium
---

# Miniport adapter OID request serialization

All OID requests to a miniport adapter are serialized by NDIS except for [direct OID requests](miniport-adapter-direct-oid-requests.md), which were designed not be serialized. A miniport adapter will not receive a new OID request until any pending request is completed. Therefore, miniport adapters must complete OIDs promptly.

>[!NOTE]
> We recommend completing an OID request in less than 1000ms, or 1 second, so the user will not notice any delay in performance. For specific information about timing OID requests, see the [NdisTimedOidComplete](https://msdn.microsoft.com/library/windows/hardware/dn305120) Driver Verifier rule.

One exception to this OID serialization rule is for Wi-Fi miniport adapters that use WDI, which may see a second OID request if they take too long to complete the previous OID. The following example explains what happens in this situation:

1. The first OID request is passed to the WDI miniport adapter.
2. The NIC does not respond to the OID within the time limit specified by the driver.
3. WDI calls the driver's [MINIPORT_WDI_ADAPTER_HANG_DIAGNOSE](https://msdn.microsoft.com/library/windows/hardware/mt297558) callback function to collect diagnostic data about the NIC.
4. The first OID is no longer considered to block serialization. This means the WDI miniport adapter can now receive other OID requests, even though the first OID is serialized. However, these other OIDS are also serialized, which means the WDI miniport adapter will not pend more than 2 OIDs simultaneously (the first OID that is still hung and a second OID).

## Related topics

For more information about WDI UE hang detection, see [UE hang detection: Steps 1-14](https://msdn.microsoft.com/windows/hardware/drivers/network/wdi-ue-hang-detection--step-1-to-step-14).

For more information about OID requests in NDIS, see [Simplifying your OID request handler](https://go.microsoft.com/fwlink/p/?linkid=846658) on the NDIS blog.

