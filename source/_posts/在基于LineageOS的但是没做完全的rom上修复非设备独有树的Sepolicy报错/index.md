---
title: 在基于LineageOS的但是没做完全的rom上修复非设备独有树的Sepolicy报错
date: 2024-07-13 09:36:05
---

现在的Evolution X（XYZ）转向了以LineageOS为底层。由于进行一大批自己的bringup，导致了默认环境变量配置出现了许多意外。上代码：

```Makefile
ifneq ($(LINEAGE_BUILD),)
ifneq ($(wildcard device/lineage/sepolicy/common/sepolicy.mk),)
## We need to be sure the global selinux policies are included
## last, to avoid accidental resetting by device configs
$(eval include device/lineage/sepolicy/common/sepolicy.mk)
endif
endif
```
让我们来逐条分析每行配置：
* 检测了LINEAGE_BUILD环境变量，如果已配置为true则继续进行下面的配置
* 检测了是否存在`device/lineage/sepolicy/common/sepolicy.mk`。如果有则继续向下执行
* 两行注释，不用管
* 上面的条件全部符合，才能include这一Makefile。
很明显，有两步判断，任何一步未成立均会影响构建，导致出现离谱的sepolicy，例如：
```
[  0% 407/61912] //system/sepolicy:vendor_sepolicy.cil.raw Building cil for vendor_sepolicy.cil.raw [common]
FAILED: out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw
out/host/linux-x86/bin/checkpolicy -C -M -c 30 -o out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw out/soong/.intermediates/system/sepolicy/vendor_sepolicy.conf/android_common/fajita/vendor_sepolicy.conf &amp;&amp; out/host/linux-x86/bin/build_sepolicy filter_out -f out/soong/.intermediates/system/sepolicy/reqd_policy_mask.cil/android_common/reqd_policy_mask.cil -t out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw # hash of input list: 552a8aec8b9bb85ede9e6c4edaad35afddf3a8553de6c319f9d84601ba2ce690
device/lineage/sepolicy/qcom/vendor/hal_lineage_livedisplay_qti.te:2:ERROR 'attribute hal_lineage_livedisplay_server is not declared' at token ';' on line 84520:
typeattribute hal_lineage_livedisplay_qti hal_lineage_livedisplay_server;
#line 2
checkpolicy:  error(s) encountered while parsing configuration
```
再如：
```
[ 62% 103975/165586] //system/sepolicy:vendor_sepolicy.cil.raw Building cil for vendor_sepolicy.cil.raw [common]
FAILED: out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw
out/host/linux-x86/bin/checkpolicy -C -M -c 30 -o out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw out/soong/.intermediates/system/sepolicy/vendor_sepolicy.conf/android_common/fajita/vendor_sepolicy.conf &amp;&amp; out/host/linux-x86/bin/build_sepolicy filter_out -f out/soong/.intermediates/system/sepolicy/reqd_policy_mask.cil/android_common/reqd_policy_mask.cil -t out/soong/.intermediates/system/sepolicy/vendor_sepolicy.cil.raw/android_common/fajita/vendor_sepolicy.cil.raw # hash of input list: 552a8aec8b9bb85ede9e6c4edaad35afddf3a8553de6c319f9d84601ba2ce690
device/lineage/sepolicy/qcom/vendor/hal_lineage_health_default.te:2:ERROR 'syntax error' at token 'rw_dir_file' on line 84515:
#line 1 "device/lineage/sepolicy/qcom/vendor/hal_lineage_health_default.te"
rw_dir_file(hal_lineage_health_default, sysfs_battery_supply)
checkpolicy:  error(s) encountered while parsing configuration
```

## 解决办法：
* 执行编译前执行：
```
export LINEAGE_BUILD=true
```
或者加入你的common.mk/BoardConfig.mk。

* 等待上游修复

## :3
