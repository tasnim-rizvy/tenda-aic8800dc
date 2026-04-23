# Tenda-AIC8800DC
FIXED DRIVER for Tenda U2 AX300

This driver has been patched for Linux kernel 6.18+ compatibility.
Tested on: Linux 6.18.12+kali-amd64

## Build & Installation

```bash
# Install the driver
sudo dpkg -i ax300.deb
```

## Verify Installation

```bash
# Check if driver is loaded
lsmod | grep aic

# Check for wireless interface
ip a
iw dev

# Bring interface up
sudo ip link set wlan0 up

# Scan for networks
sudo iw dev wlan0 scan | grep SSID
```

## Kernel 6.18 Compatibility Fixes

The following API changes in kernel 6.18 required modifications:

### 1. MODULE_IMPORT_NS
The MODULE_IMPORT_NS macro was removed in kernel 6.18.

```c
#if LINUX_VERSION_CODE >= KERNEL_VERSION(5, 4, 0) && LINUX_VERSION_CODE < KERNEL_VERSION(6, 18, 0)
MODULE_IMPORT_NS(VFS_internal_I_am_really_a_filesystem_and_am_NOT_a_driver);
#endif
```

### 2. Timer API Changes (from_timer, del_timer, del_timer_sync)
- `from_timer()` replaced with `container_of()`
- `del_timer()` replaced with `timer_delete()`
- `del_timer_sync()` replaced with `timer_delete_sync()`

```c
#if LINUX_VERSION_CODE >= KERNEL_VERSION(6, 18, 0)
    rwnx_hw = container_of(t, struct rwnx_hw, p2p_alive_timer);
#else
    rwnx_hw = from_timer(rwnx_hw, t, p2p_alive_timer);
#endif
```

### 3. cfg80211_rx_spurious_frame()
Added new `u32 freq` argument in kernel 6.18.

```c
#if LINUX_VERSION_CODE >= KERNEL_VERSION(6, 18, 0)
    cfg80211_rx_spurious_frame(ndev, addr, 0, GFP_ATOMIC);
#else
    cfg80211_rx_spurious_frame(ndev, addr, GFP_ATOMIC);
#endif
```

### 4. cfg80211_rx_unexpected_4addr_frame()
Same change as above - new freq argument.

### 5. cfg80211_cac_event()
Added new 5th argument in kernel 6.18.

```c
#if LINUX_VERSION_CODE >= KERNEL_VERSION(6, 18, 0)
    cfg80211_cac_event(ndev, &chan_def, NL80211_RADAR_CAC_FINISHED, 0, GFP_KERNEL);
#else
    cfg80211_cac_event(ndev, &chan_def, NL80211_RADAR_CAC_FINISHED, GFP_KERNEL);
#endif
```

### 6. cfg80211_ops Callbacks (Kernel 6.18 new signatures)
- `set_monitor_channel`: Added `struct net_device *` parameter
- `set_wiphy_params`: Added `int antennas` parameter
- `set_tx_power`: Added `int idx` parameter
- `start_radar_detection`: Added `int count` parameter

## Credits

Original driver for kernels 3.10-6.8.

Thanks to https://github.com/Z0mbl03 for the initial fix for kernels 5.19+.