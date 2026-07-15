# Goodix Touchscreen Makefile
# This Makefile is only for tempory compiling use
# Use xxxproject/Makefile-release when release driver
#
# core layer
ccflags-$(CONFIG_DRM_MEDIATEK) += -I$(srctree)/drivers/gpu/drm/mediatek/
ccflags-y += -Wno-vla

obj-y += goodix_ts_gt9886.o
goodix_ts_gt9886-y := goodix_ts_i2c.o goodix_ts_core.o goodix_cfg_bin.o goodix_ts_tools.o goodix_gtx8_update.o
