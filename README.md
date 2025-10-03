# Bypassing-AT-T-s-BGW320-500-Gateway-with-Unifi-Cloud-Gateway-UCG-Fiber-Using-wpa_supplicant
This guide details my process for bypassing the AT&amp;T BGW320-500 gateway (the large, bulky Optical Network Terminal or ONT box) and replacing it with a Unifi Cloud Gateway (UCG) Fiber using a wpa_supplicant bypass method. It's specifically tailored for the AT&amp;T BGW320-500 model.
A huge thank you to the 8311 Discord community, especially @jack2333, @Dong Vu, and @kaabob, for their guidance. To give back, I'm sharing this polished guide based on my experiences.
# Bypassing AT&T's BGW320-500 Gateway with Unifi Cloud Gateway (UCG) Fiber Using `wpa_supplicant`

This guide details the process for bypassing the AT&T BGW320-500 gateway (the large Optical Network Terminal or ONT box) and replacing it with a Unifi Cloud Gateway (UCG) Fiber using a `wpa_supplicant` bypass method. It is specifically tailored for the AT&T BGW320-500 model.

A huge thank you to the 8311 Discord community, especially @jack2333, @Dong Vu, and @kaabob, for their guidance. To give back, I’m sharing this polished guide based on my experiences.

## Resources
The three main resources I followed:
- [PON Madness - AT&T GPON ONT Cloning/Bypass](https://docs.google.com/document/d/1gcT0sJKLmV816LK0lROCoywk9lXbPQ7l_4jhzGIgoTo/edit?tab=t.0#heading=h.5zuhlpp9nnh6) (firmware source)
- [Unifi-gateway-wpa-supplicant by Evie Lau](https://github.com/evie-lau/unifi-gateway-wpa-supplicant)
- [Certs Extraction Repo by 0x888e](https://github.com/0x888e/certs)

## Important Notes
- This assumes you have a 1310 nm wavelength GPON connection. Verify this first: If your setup uses 1310 nm, it’s GPON-compatible.
- Fiber connectors come in two standards: APC (green, angled) or UPC (blue, flat). I used APC; confirm yours when purchasing hardware.
- **Proceed at your own risk**—this involves firmware flashing and network modifications that could void warranties or disrupt service.

## Prerequisites and Hardware
Before starting, gather these items:
- **SFP Module**: ISZO XPON Stick (Model: LL-XS2510). The Amazon link shows UPC, but they shipped APC.  
  [Amazon Link]([https://www.amazon.com/ISZO-XPON-Stick](https://www.amazon.com/iszo-Support-Modify-Network-Converter/dp/B0BZPZNKJ6/ref=sr_1_2?crid=1MPOAUZTXX6Z7&dib=eyJ2IjoiMSJ9.o2dtiq_HWtz9MxS9ZlPvbmlJYutTI0FaEOpW8E1YnHFiVmyYlcaTDfy1wFGYvipziQiPU1X4mQi0vYip6Y7XSVmEAB0MFjV0iMlGd6lgpzn4xyrXu6HRzDaixBXyRfle-voY_m-6YecsMr1uiATO_w.h_6WBEN9vwvmgm3rBfTQN2522uoJ0W8USJYI-nm7-LE&dib_tag=se&keywords=iszo+gpon&qid=1759467499&sprefix=iszo+gpon%2Caps%2C361&sr=8-2)
- **Media Converter**: TP-Link MC220L Gigabit Media Converter (for easy access to the SFP module’s settings without direct UCG integration).  
  [Amazon Link]([https://www.amazon.com/TP-Link-MC220L-Gigabit-Media-Converter](https://www.amazon.com/TP-Link-Ethernet-Converter-Supporting-MC220L/dp/B003CFATL0/ref=sr_1_1?crid=137QG05LDZW0V&dib=eyJ2IjoiMSJ9.UblUIYiFc9US9qVT7_drQn5G8j-QlFuDuvSa1P3mNRWXpMxfhY9t9FYDn3kylGGmgyUrVkrMnnDwVIAwRibckOf-TRKXlpznjBCPbZmAVlKXD086hrJ-8zSZnkdSteYw7r_xHsTOlzPgX1m44HKMk8UERH2uocew3E44i7VAMwuDDbWMdS9ClONF4q5FnFGzhjXxPpSZ1-Ys9sDis-UEDbnD9hJ9ut7eROMXK6Hhczw.4YP8zCv0rjjrTnPdWP8G9NEh1yrZV_7ukOgfkQLa_7o&dib_tag=se&keywords=TP-Link%2BMC220L&qid=1759467522&sprefix=tp-link%2Bmc220l%2Caps%2C164&sr=8-1&th=1)
- **Firmware Download**: `M110_sfp_ODI_220923FS.tar` from the RTL960x repo.  
  [Download Link](https://github.com/rajkosto/RTL960x/blob/main/Firmware/DFP-34X-2C2/M110_sfp_ODI_220923FS.tar)
- **Additional Requirements**:
  - A computer/laptop for configuration.
  - PuTTY (or similar) for Telnet access.
  - Access to the BGW320-500’s web UI for details like ONT ID and firmware version.

## Step 1: Flashing Firmware on the SFP Module
1. Insert the SFP module into the media converter and connect it to your computer via Ethernet.
2. Set your computer’s IP to `192.168.1.10` (subnet mask: `255.255.255.0`) to match the module’s subnet.
3. In Command Prompt, ping `192.168.1.1` to confirm connectivity.
4. Open a web browser (try incognito mode if issues arise) and navigate to `http://192.168.1.1`.
5. Flash the downloaded firmware (`M110_sfp_ODI_220923FS.tar`). Flash it **twice** (the module stores two images).
6. Reset the module to default settings.
7. In VLAN settings, set to **Manual Mode > Transparent** (to pass all traffic untagged).

## Step 2: Configuring the SFP Module via Telnet
1. Install PuTTY and connect to `192.168.1.1` via Telnet.
2. Run the following commands (based on PON Madness for BGW320-500):
   ```
   flash set OMCC_VER 160
   flash set GPON_PLOAM_FORMAT 1
   flash set GPON_PLOAM_PASSWD 44454641554c54000000
   flash set OMCI_OLT_MODE 21
   flash set OMCI_FAKE_OK 1
   flash set VLAN_CFG_TYPE 1
   flash set VLAN_MANU_MODE 0
   flash set PON_VENDOR_ID HUMA
   flash set GPON_ONU_MODEL iONT320500G
   flash set GPON_SN HUMAxxxxxxxx  # Replace with your ONT ID from the bottom of the BGW320-500
   flash set HW_HWVER BGW320-500_2.1
   flash set OMCI_SW_VER1 BGW320_3.20.5  # Replace with your firmware version from BGW320 web UI
   flash set OMCI_SW_VER2 BGW320_3.20.5  # Same as above
   reboot
   ```
   These are the only two that needs modifying:
   - **GPON_SN**: ONT ID from the BGW320-500 bottom label (e.g., `HUMAxxxxxxxx`).
   - **OMCI_SW_VER1/2**: Firmware version from BGW320 System Information page.

## Step 3: Verifying O5 Status and Extracting VLAN ID
1. After reboot, reconnect via Telnet and plug in the fiber cable.
2. Check ONU state:
   ```
   diag gpon get onu-state
   ```
   It should show `O5`. If not, review your settings—handshake failed.
3. Get VLAN ID:
   ```
   omcicli mib get 84
   ```
   Example output:
   ```
   EntityID: 0x1102
   FilterTbl[0]: PRI 0, CFI 0, VID 722
   FwdOp: 0x10
   NumOfEntries: 1
   ```
   Here, `722` is the VLAN ID assigned by the ISP.
4. Check for EAP Authentication:
   ```
   omcicli mib get 171
   ```
   - 5 rules (indexes 0–4): EAP required.
   - 7 rules (indexes 0–6): No EAP; assign VLAN ID directly on WAN port for internet.
5. You can now remove the media converter and insert the SFP into the UCG Fiber Gateway (Port 7, `eth6`).

## Step 4: Extracting Certificates from BGW320-500
1. Follow the [certs repo guide](https://github.com/0x888e/certs).
2. Download firmware from the provided source.
3. **Safety First**: Unplug the fiber cable.
4. Downgrade to `spTurquoise320-500_3.17.5_dnvpnP_021_sec` (device will boot loop).
5. Quickly patch to `spTurquoise320-500_3.18.1_sec` during the loop window.
6. Run `download.py` from the certs repo to extract certificates.
7. Ignore extra firmwares in the ZIP (e.g., `4.24.6_sec`, `6.28.7_sec`).
8. Reconnect fiber after extraction—the ISP will auto-upgrade.

## Step 5: Setting Up `wpa_supplicant` on UCG Fiber Gateway
1. Follow [Evie Lau’s guide](https://github.com/evie-lau/unifi-gateway-wpa-supplicant) up to the “Spoof MAC Address” section.
2. In UCG web UI, assign the VLAN ID (from Step 3) to the WAN port.
3. Assign the MAC address found on the bottom of BGW320-500.
4. Enable SSH: Control Plane > Console.
5. SSH into the gateway: `ssh root@your_gateway_ip`.
6. Test setup:
   ```
   ip link add link eth6 name eth6.0 type vlan id 0
   wpa_supplicant -i eth6.0 -D wired -c /etc/wpa_supplicant/wpa_supplicant.conf -ddd
   ```
   This creates a priority-tagged subinterface (`eth6.0`). Use `-ddd` for debug output. Press `Ctrl+C` to exit.
7. If successful, proceed to persistence.

## Step 6: Making Configurations Persistent
### VLAN Setup Script
Create a systemd service file at `/etc/systemd/system/eth6.0-vlan.service` to configure the VLAN interface `eth6.0` before the network interface is brought up.

```bash
cat << EOF > /etc/systemd/system/eth6.0-vlan.service
[Unit]
Description=Configure VLAN eth6.0
Before=wpa_supplicant-wired@eth6.0.service
After=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/sbin/ip link add link eth6 name eth6.0 type vlan id 0
ExecStart=/sbin/ip link set dev eth6.0 up
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF
```

### Steps to Apply
1. Reload systemd configuration:
   ```
   systemctl daemon-reload
   ```
2. Restart the VLAN service:
   ```
   systemctl restart eth6.0-vlan.service
   ```
3. Check the service status:
   ```
   systemctl status eth6.0-vlan.service
   ```
4. **Note**: If the `eth6.0` interface is already up, the script may output an error, which can be safely ignored.

### `wpa_supplicant` Auto-Start
1. Rename config:
   ```
   mv /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant-wired-eth6.0.conf
   ```
2. Set permissions:
   ```
   chmod 600 /etc/wpa_supplicant/wpa_supplicant-wired-eth6.0.conf
   ```
3. Restart and enable service:
   ```
   systemctl daemon-reload
   systemctl start wpa_supplicant-wired@eth6.0
   systemctl enable wpa_supplicant-wired@eth6.0
   systemctl status wpa_supplicant-wired@eth6.0
   ```
4. For surviving firmware updates, follow Evie Lau’s “Survive Firmware Updates” section (replace `eth1` with `eth6` for UCG).

## Conclusion
With these steps, your setup should now bypass the BGW320-500 and run internet through the UCG Fiber Gateway. If issues arise, double-check VLAN, certs, and EAP rules. Thanks again to the 8311 Discord community for making this possible!
