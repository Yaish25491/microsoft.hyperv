##### SUMMARY
This PR addresses **GitHub Issue #20** by significantly expanding the boot management capabilities of the `hv_vm_boot` module to accurately support both Generation 1 and Generation 2 Virtual Machines.

**Design Decisions:**
- **Generation 1 Enhancements:** The original `startup_order` parameter was missing valid device choices. It has been updated to include `NetworkAdapter` and `VHD`, allowing full configuration of legacy BIOS environments.
- **Generation 2 (UEFI) Support:** A brand new `boot_order` parameter was introduced exclusively for Generation 2 VMs. Because Gen 2 VMs use `Set-VMFirmware` (and not `Set-VMBios`), they require a fundamentally different approach. 
- **Robust Idempotency:** Hyper-V often populates the firmware boot order with multiple devices of the same type (e.g., multiple "EFI Network" adapters). The `boot_order` logic intelligently maps simplified Ansible inputs (`Network`, `DVD`, `SCSI`, `File`) to the corresponding `VMBootSource` objects and uses their unique `FirmwarePath` for precise change detection.
- **Validation:** The integration test suite (`tests.yml`) has been updated to spin up temporary virtual hardware and verify these new boot sequencing features across both VM generations.

Fixes #20 

##### ISSUE TYPE
- Feature Pull Request
- Bugfix Pull Request

##### COMPONENT NAME
`plugins/modules/hv_vm_boot`

##### ADDITIONAL INFORMATION
**Example Usage (Generation 1):**
```yaml
- name: Set Startup Order for a Gen1 VM
  microsoft.hyperv.hv_vm_boot:
    name: Gen1LegacyVM
    startup_order:
      - VHD
      - CD
      - IDE
```

**Example Usage (Generation 2):**
```yaml
- name: Configure Boot Order for a Gen2 VM
  microsoft.hyperv.hv_vm_boot:
    name: Gen2VM
    boot_order:
      - SCSI
      - DVD
      - Network
```