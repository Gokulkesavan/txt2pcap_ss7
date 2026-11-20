# SS7 hexdump to PCAP

Converts Huawei UGC/MSOFTX PTMF-derived text exports into a Wireshark-readable Ethernet PCAP.

## Supported records

- SIP messages from `TRC_MI_FROM_SIP` / `TRC_MI_TO_SIP`, including SIP stored as ASCII hex bytes in the PTMF export.
- Existing M3UA, MTP3, SCCP and SCCP-user trace formats supported by the converter.
- Huawei network-side SCCP UDT records from `TRC_MI_SCCPDPC_MSG`. The converter skips the 12-byte PTMF trace prefix, takes the SCCP UDT beginning with `0x09`, and passes its TCAP BER user data through the existing M3UA/SCTP/IPv4/Ethernet packet builder.

## TCAP / CAP / INAP payload handling

The TCAP BER payload is preserved byte-for-byte inside SCCP user data. This allows Wireshark to dissect TCAP and, where the application-context OID and operation definitions are recognized, the corresponding CAP/INAP application layer.

The converter uses `TRC_MI_SCCPDPC_MSG` as the network-facing packet source. It does not separately emit `TRC_MI_TCAPUSER_MSG`, `TRC_MI_FROM_CAP`, `TRC_MI_TO_CAP`, `TRC_MI_TCAPSCCP_MSG`, or `TRC_MI_SCCPUSER_MSG` primitive/interface views as additional packets, avoiding duplicate representations of the same exchange.

## Reconstruction notes

- The existing wrapper values remain in use: M3UA OPC/DPC `1/2`, SCTP ports `2905/2905`, IPv4 addresses `10.0.0.1/10.0.0.2`, and synthetic Ethernet MAC addresses.
- This is packet reconstruction from a PTMF trace, not a bit-for-bit capture of the original SS7 link. Original MTP3 point codes and link-layer addressing are not currently reconstructed.
- The converter preserves TCAP/CAP/INAP BER bytes; it does not translate INAP operations or synthesize missing TCAP components.

## Use

Open the page, select the Huawei-derived `.txt` export, and choose **Convert → PCAP**. Open the downloaded file in Wireshark and use:

```
sip || m3ua || sccp || tcap || gsm_map
```
