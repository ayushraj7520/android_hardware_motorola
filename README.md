# Motorola Hardware HALs (`lyriq` / `lineage-22.1`)

Customized Motorola hardware repository for **Motorola Edge 40 (`lyriq`)** on **LineageOS 22.1**.

## Components

- **`aidl/fingerprint`**: Open-source Motorola AIDL Fingerprint HAL (`android.hardware.biometrics.fingerprint-service.motorola`) supporting Egis RBS / Goodix optical UDFPS sensors (`persist.vendor.fingerprint.type=udfps_optical`) and interfacing with `libudfpshandler`.
- **`udfps`**: Header definitions and default UDFPS handler glue (`UdfpsHandler.h`).
- **`sepolicy` & `vintf`**: SELinux rules and VINTF declarations for Motorola fingerprint service.

---

## UDFPS HBM Fix (`mediatek-drm.ko` IOCTL Offsets in `device/motorola/lyriq/udfps`)

To enable High Brightness Mode (HBM) during fingerprint authentication on the Motorola Edge 40 (`lyriq`), proprietary DRM ioctl offsets were reverse-engineered from Motorola's `mediatek-drm.ko` and `dsi-panel-mot-boe-nt37701a-655-fhdp-video-144hz.ko` and added to `device/motorola/lyriq/udfps/mediatek_drm.h`:

```c
/* panel feature */
typedef enum {
    PARAM_HBM = 0,
    PARAM_CABC,
    PARAM_ACL,
    PARAM_DC,
    POWER_MODE_MAX_NUM,
    PARAM_COLOR = POWER_MODE_MAX_NUM,
    PARAM_MAX_NUM
} paramId_t;

struct panel_param_info {
    paramId_t param_idx; // 0x0: PARAM_HBM (0)
    uint32_t value;      // 0x4: 0 = OFF, 1 = Sunlight HBM, 2 = FOD HBM
};

#define DRM_SET_PANEL_FEATURE        (DRM_COMMAND_END - 2) /* 0x9E -> 0xC008649E */
#define DRM_IOCTL_SET_PANEL_FEATURE  DRM_IOWR(DRM_SET_PANEL_FEATURE, struct panel_param_info)
#define DRM_GET_PANEL_FEATURE        (DRM_COMMAND_END - 3) /* 0x9D -> 0xC008649D */
#define DRM_IOCTL_GET_PANEL_FEATURE  DRM_IOWR(DRM_GET_PANEL_FEATURE, struct panel_param_info)
```

In `UdfpsHandler.cpp` (`libudfpshandler`):
- **`onFingerDown`**: Calls `ioctl(fd, DRM_IOCTL_SET_PANEL_FEATURE, &param_info)` on `/dev/dri/card0` with `{ PARAM_HBM, 2 }` (FOD Local HBM ON) and sends `sendFodEvent(0)` (`FINGER_DOWN`) with a non-empty dummy byte vector (`{ 0 }`) to `com.motorola.hardware.biometric.fingerprint.IMotoFingerPrint/default`.
- **`onFingerUp` / `onAuthenticationSucceeded` / `cancel`**: Calls `DRM_IOCTL_SET_PANEL_FEATURE` with `{ PARAM_HBM, 0 }` (HBM OFF) and sends `sendFodEvent(1)` (`FINGER_UP`).
