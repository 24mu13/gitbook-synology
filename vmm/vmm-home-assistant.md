# Supervised Home Assistant

- Download the latest version of the [VMware ESXi/vSphere](https://www.home-assistant.io/installation/alternative) in .ova format
- Open **Virtual Machine Manager**
- Click on **Virtual Machine** -> **Import** and select the .ova file
- Select the storage
- Leave default values (??)
- _Configure Network_: make sure to select **e1000** model on gear icon for advanced settings
- _Other Settings_
  - Autostart: Yes
  - Firmware: UEFI
- Power on the VM on the final summary page

## Assumptions

- You have installed **Virtual Machine Manager**

## References

- [How to Set Up Home Assistant on a Synology NAS (Supervised)](https://www.wundertech.net/how-to-set-up-home-assistant-supervised-on-a-synology-nas/)
