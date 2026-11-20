# SS7 hexdump to PCAP

Converts Huawei UGC/MSOFTX PTMF-derived text exports into a Wireshark-readable Ethernet PCAP.

## Supported records

- SIP messages from `TRC_MI_FROM_SIP` / `TRC_MI_TO_SIP`, including SIP stored as ASCII hex bytes in the PTMF export.
- Existing M3UA, MTP3, SCCP and SCCP-user trace formats supported by the converter.
- Huawei network-side SCCP UDT records from `TRC_MI_SCCPDPC_MSG`. The converter skips the 12-byte PTMF trace prefix, takes the SCCP UDT beginning with `0x09`, and passes its TCAP BER user data through the existing M3UA/SCTP/IPv4/Ethernet packet builder.

## TCAP / CAP / INAP payload handling

Huawei PTMF places the hex data on the same line as `[Msg Body]`. The parser retains this inline data (and also supports payload lines following the marker); without this, the packet builder receives empty bodies.

The TCAP BER payload is preserved byte-for-byte inside SCCP user data. The converter identifies the INAP CS2 application-context OID `0.0.17.773.5.1.1.1` (`06 07 00 11 86 05 01 01 01`) and tracks the TCAP OTID/DTID values so subsequent Continue, End and Abort messages in the same dialogue are also classified as INAP. The OID count is therefore not incorrectly used as the total INAP packet count. Wireshark can then dissect TCAP and the applicable INAP/CAP layer when its protocol dissectors recognize the context and operation definitions.

The converter uses `TRC_MI_SCCPDPC_MSG` as the network-facing packet source. It does not separately emit `TRC_MI_TCAPUSER_MSG`, `TRC_MI_FROM_CAP`, `TRC_MI_TO_CAP`, `TRC_MI_TCAPSCCP_MSG`, or `TRC_MI_SCCPUSER_MSG` primitive/interface views as additional packets, avoiding duplicate representations of the same exchange.

## Reconstruction notes

- For `TRC_MI_SCCPDPC_MSG`, the 24-bit OPC and DPC are read from the PTMF prefix and placed into the M3UA Protocol Data parameter. Stable synthetic IPv4 addresses are derived from those point codes, preserving source/destination direction across packets. The SLS is also retained; SI is set to SCCP (3) and NI is set to 0, matching the corresponding captured M3UA linkset record.
- SCTP ports remain synthetic `2905/2905`, and Ethernet MAC addresses are synthetic. This is packet reconstruction from a PTMF trace, not a bit-for-bit capture of the original SS7 link.
- The converter preserves TCAP/CAP/INAP BER bytes; it does not translate INAP operations or synthesize missing TCAP components.

## Use

Open the page, select the Huawei-derived `.txt` export, and choose **Convert → PCAP**. The conversion log includes TCAP packet count and the count of packets associated with an INAP CS2 dialogue, including subsequent messages without a repeated application-context OID.

Open the downloaded file in Wireshark and use:

```
sip || m3ua || sccp || tcap || inap
```
