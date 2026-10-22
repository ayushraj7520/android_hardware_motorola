# Motorola Hardware HALs (`lyriq` / `lineage-22.1`)

Customized Motorola hardware repository for **Motorola Edge 40 (`lyriq`)** on **LineageOS 22.1**.

## Components

- **`aidl/fingerprint`**: Open-source Motorola AIDL Fingerprint HAL (`android.hardware.biometrics.fingerprint-service.motorola`) supporting Egis RBS / Goodix optical UDFPS sensors (`persist.vendor.fingerprint.type=udfps_optical`) and interfacing with `libudfpshandler`.
- **`udfps`**: Header definitions and default UDFPS handler glue (`UdfpsHandler.h`).
- **`sepolicy` & `vintf`**: SELinux rules and VINTF declarations for Motorola fingerprint service.
