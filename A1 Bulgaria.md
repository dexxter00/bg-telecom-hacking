# A1 BG GPON onto proprietary equipment

## How it works?
This ISP is using Huawei HG8145V5 ONT. 
Authentication with OLT is currently based only on **Serial Number** and nothing more.
Some other parameters described below are being taken into account for successful authentication ( O5 status ).

## Hardware needed
You will need some equipment prior to success.
- Router
- GPON SFP stick - highly recommend ODI DFP-34X-2C2 XPON ( [Ali](https://www.aliexpress.com/item/1005003515662920.html) )
- Gigabit SFP port to plug the stick - may be router with SFP port, a switch, PCI-Express card or external SFP to Ethernet converter(cheap and easy option).

## Gathering info
First things first - a serial number. Serial number is located on the stock device's label. It consists of 16 characters and looks something like that **4857544312345678**
First 8 of them are always the same for Huawei. Converted to ASCII **48575443** -> **HWTC**.

Other parameters:
- ONU Model
- Vendor ID
- HW Version
- SW Version

These parameters can be easily seen at the web GUI of the device itself.
At this moment I DO NOT recommend you to reset your device to check this info, just use the one provided below.

| `GPON_ONU_MODEL` | `GPON_SN`    |`PON_VENDOR_ID` | `HW_HWVER`      | `OMCI_SW_VER`    |
|------------------|--------------|----------------|-----------------|------------------|
| HG8145V5         | HWTC12345678 | HWTC           | 15AD.A          | V5R019C10S270    |

MAC address is not incorporated into GPON frames, so is not needed for authentication. MAC addr is only used for DHCP server to know who you are (OLT terminal and DHCP/billing server are separated).
Since this device uses different MAC addresses for different interfaces, another trick is to call the provider and make them do a so-called "bridge". This way, their system will save MAC address of your router.

## Connect to SFP module
These modules are not just optical transcievers. Specific for them is that they have some sort of Linux OS running inside them. And so - a web interface and a telnet/SSH connection. Highly recommend not to touch the Web GUI. Telnet is a good way for configuration.

Default IP address is **192.168.1.1**, so set your home network to something else(ex. 192.168.5.1). Put SFP interface inside WAN firewall group of your router and you should be able to connect.

## Configuration
Login to Telnet and copy/paste these one by one. Do not forget to change **GPON_SN** according to your label!!
```
flash set flash set OMCI_OLT_MODE 21
flash set GPON_ONU_MODEL HG8145V5
flash set GPON_SN HWTC********
flash set PON_VENDOR_ID HWTC
flash set HW_HWVER 15AD.A
flash set OMCI_SW_VER1 V5R019C10S270
flash set OMCI_SW_VER2 V5R019C10S270
flash set OMCI_FAKE_OK 1
flash set flash set OMCI_OLT_MODE 21
flash set HW_CWMP_MANUFACTURER 'Huawei Technologies Co., Ltd'
flash set OMCI_TM_OPT 1
reboot
```

## Further configuration
A1 OLT provisions you a list of parameters after successful authentication. One of them being GEM port(s) and VLANs.

ODI stick will privision VLANs and bridge them accordingly to SFP port.
- Internet is at VLAN 555
- VoIP is at VLAN 666
- IPTV is at VLAN 777

Setup interface on your router so WAN interface uses tagged VLAN (ethX.555) and DHCP protocol.

That's it. Do a speedtest and enjoy!