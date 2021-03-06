---
title: "Android Product Buildvars"
description: "Some gotchas"
date: 2019-03-20T21:19:16+01:00
draft: false
---

The following build variables can be set in files inherited from
`AndroidProducts.mk` or `$PRODUCT.mk`, e.g. `aosp_f8331.mk`.
Some of them become read-only to kati (part of the build system) and setting
their value at a later stage - e.g. in `BoardConfig.mk` or inherited files - is
ignored.

`build/make/core/product.mk`
```make
_product_var_list := \
    PRODUCT_NAME \
    PRODUCT_MODEL \
    PRODUCT_LOCALES \
    PRODUCT_AAPT_CONFIG \
    PRODUCT_AAPT_PREF_CONFIG \
    PRODUCT_AAPT_PREBUILT_DPI \
    PRODUCT_PACKAGES \
    PRODUCT_PACKAGES_DEBUG \
    PRODUCT_PACKAGES_ENG \
    PRODUCT_PACKAGES_TESTS \
    PRODUCT_DEVICE \
    PRODUCT_MANUFACTURER \
    PRODUCT_BRAND \
    PRODUCT_PROPERTY_OVERRIDES \
    PRODUCT_DEFAULT_PROPERTY_OVERRIDES \
    PRODUCT_PRODUCT_PROPERTIES \
    PRODUCT_CHARACTERISTICS \
    PRODUCT_COPY_FILES \
    PRODUCT_OTA_PUBLIC_KEYS \
    PRODUCT_EXTRA_RECOVERY_KEYS \
    PRODUCT_PACKAGE_OVERLAYS \
    DEVICE_PACKAGE_OVERLAYS \
    PRODUCT_ENFORCE_RRO_EXCLUDED_OVERLAYS \
    PRODUCT_ENFORCE_RRO_TARGETS \
    PRODUCT_SDK_ATREE_FILES \
    PRODUCT_SDK_ADDON_NAME \
    PRODUCT_SDK_ADDON_COPY_FILES \
    PRODUCT_SDK_ADDON_COPY_MODULES \
    PRODUCT_SDK_ADDON_DOC_MODULES \
    PRODUCT_SDK_ADDON_SYS_IMG_SOURCE_PROP \
    PRODUCT_SOONG_NAMESPACES \
    PRODUCT_DEFAULT_WIFI_CHANNELS \
    PRODUCT_DEFAULT_DEV_CERTIFICATE \
    PRODUCT_RESTRICT_VENDOR_FILES \
    PRODUCT_VENDOR_KERNEL_HEADERS \
    PRODUCT_BOOT_JARS \
    PRODUCT_SUPPORTS_BOOT_SIGNER \
    PRODUCT_SUPPORTS_VBOOT \
    PRODUCT_SUPPORTS_VERITY \
    PRODUCT_SUPPORTS_VERITY_FEC \
    PRODUCT_OEM_PROPERTIES \
    PRODUCT_SYSTEM_DEFAULT_PROPERTIES \
    PRODUCT_SYSTEM_PROPERTY_BLACKLIST \
    PRODUCT_VENDOR_PROPERTY_BLACKLIST \
    PRODUCT_SYSTEM_SERVER_APPS \
    PRODUCT_SYSTEM_SERVER_JARS \
    PRODUCT_ALWAYS_PREOPT_EXTRACTED_APK \
    PRODUCT_DEXPREOPT_SPEED_APPS \
    PRODUCT_LOADED_BY_PRIVILEGED_MODULES \
    PRODUCT_VBOOT_SIGNING_KEY \
    PRODUCT_VBOOT_SIGNING_SUBKEY \
    PRODUCT_VERITY_SIGNING_KEY \
    PRODUCT_SYSTEM_VERITY_PARTITION \
    PRODUCT_VENDOR_VERITY_PARTITION \
    PRODUCT_PRODUCT_VERITY_PARTITION \
    PRODUCT_SYSTEM_SERVER_DEBUG_INFO \
    PRODUCT_OTHER_JAVA_DEBUG_INFO \
    PRODUCT_DEX_PREOPT_MODULE_CONFIGS \
    PRODUCT_DEX_PREOPT_DEFAULT_COMPILER_FILTER \
    PRODUCT_DEX_PREOPT_DEFAULT_FLAGS \
    PRODUCT_DEX_PREOPT_BOOT_FLAGS \
    PRODUCT_DEX_PREOPT_PROFILE_DIR \
    PRODUCT_DEX_PREOPT_BOOT_IMAGE_PROFILE_LOCATION \
    PRODUCT_DEX_PREOPT_GENERATE_DM_FILES \
    PRODUCT_USE_PROFILE_FOR_BOOT_IMAGE \
    PRODUCT_SYSTEM_SERVER_COMPILER_FILTER \
    PRODUCT_SANITIZER_MODULE_CONFIGS \
    PRODUCT_SYSTEM_BASE_FS_PATH \
    PRODUCT_VENDOR_BASE_FS_PATH \
    PRODUCT_PRODUCT_BASE_FS_PATH \
    PRODUCT_SHIPPING_API_LEVEL \
    VENDOR_PRODUCT_RESTRICT_VENDOR_FILES \
    VENDOR_EXCEPTION_MODULES \
    VENDOR_EXCEPTION_PATHS \
    PRODUCT_ART_TARGET_INCLUDE_DEBUG_BUILD \
    PRODUCT_ART_USE_READ_BARRIER \
    PRODUCT_IOT \
    PRODUCT_SYSTEM_HEADROOM \
    PRODUCT_MINIMIZE_JAVA_DEBUG_INFO \
    PRODUCT_INTEGER_OVERFLOW_EXCLUDE_PATHS \
    PRODUCT_ADB_KEYS \
    PRODUCT_CFI_INCLUDE_PATHS \
    PRODUCT_CFI_EXCLUDE_PATHS \
    PRODUCT_COMPATIBLE_PROPERTY_OVERRIDE \
    PRODUCT_ACTIONABLE_COMPATIBLE_PROPERTY_DISABLE \
```

```make
# $(1): product to inherit
#
# Does three things:
#  1. Inherits all of the variables from $1.
#  2. Records the inheritance in the .INHERITS_FROM variable
#  3. Records that we've visited this node, in ALL_PRODUCTS
#
define inherit-product
  $(if $(findstring ../,$(1)),\
    $(eval np := $(call normalize-paths,$(1))),\
    $(eval np := $(strip $(1))))\
  $(foreach v,$(_product_var_list), \
      $(eval $(v) := $($(v)) $(INHERIT_TAG)$(np))) \
  $(eval inherit_var := \
      PRODUCTS.$(strip $(word 1,$(_include_stack))).INHERITS_FROM) \
  $(eval $(inherit_var) := $(sort $($(inherit_var)) $(np))) \
  $(eval inherit_var:=) \
  $(eval ALL_PRODUCTS := $(sort $(ALL_PRODUCTS) $(word 1,$(_include_stack))))
endef
```

`build/core/product_config.mk`
```
.KATI_READONLY := \
  TARGET_PRODUCT \
  TARGET_BUILD_VARIANT \
  TARGET_BUILD_APPS
.KATI_READONLY := PRODUCT_PROPERTY_OVERRIDES
.KATI_READONLY := PRODUCT_DEFAULT_PROPERTY_OVERRIDES
.KATI_READONLY := PRODUCT_SYSTEM_DEFAULT_PROPERTIES
.KATI_READONLY := PRODUCT_PRODUCT_PROPERTIES
```
