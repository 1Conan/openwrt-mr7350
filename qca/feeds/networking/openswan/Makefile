#
# Copyright (C) 2006-2013 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=openswan
PKG_VERSION:=2.6.51.3
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://www.openswan.org/download
PKG_MD5SUM:=03339eba2aff132627e9a85eddf69263

PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk

define Package/openswan/Default
  TITLE:=Openswan
  URL:=http://www.openswan.org/
endef

define Package/openswan/Default/description
 Openswan is an IPsec implementation for Linux.
endef

define Package/openswan
$(call Package/openswan/Default)
  SUBMENU:=VPN
  SECTION:=net
  CATEGORY:=Network
  TITLE+= (daemon)
  DEPENDS+= +libgmp +PACKAGE_kmod-crypto-ocf:kmod-openswan
endef

define Package/openswan/description
$(call Package/openswan/Default/description)
 This package contains the Openswan user-land daemon.
endef

define KernelPackage/openswan
$(call Package/openswan/Default)
  SUBMENU:=Network Support
  TITLE+= (kernel module)
  DEPENDS:=+PACKAGE_kmod-crypto-ocf:kmod-crypto-ocf
  FILES:=$(PKG_BUILD_DIR)/modobj*/ipsec.$(LINUX_KMOD_SUFFIX)
endef

define KernelPackage/openswan/description
$(call Package/openswan/Default/description)
 This package contains the Openswan kernel module.
endef

TARGET_CPPFLAGS += -I$(LINUX_DIR)/$(LINUX_UAPI_DIR)include -D__EXPORTED_HEADERS__

OPENSWAN_MAKEFLAGS:= \
	-C $(PKG_BUILD_DIR) \
	$(TARGET_CONFIGURE_OPTS) \
	LINUX_RELEASE="$(LINUX_RELEASE)" \
	KERNELSRC="$(LINUX_DIR)" \
	ARCH="$(LINUX_KARCH)" \
	CROSS_COMPILE="$(TARGET_CROSS)" \
	USERCOMPILE="$(TARGET_CFLAGS) -fPIE -I$(PKG_BUILD_DIR)/ports/linux/include -I$(PKG_BUILD_DIR)/linux/include -I$(PKG_BUILD_DIR)/include $(TARGET_CPPFLAGS) $(TARGET_LDFLAGS)" \
	IPSECDIR="/usr/lib/ipsec" \
	INC_USRLOCAL="/usr" \
	INC_RCDEFAULT="/etc/init.d" \
	MODPROBE="insmod" \
	DESTDIR="$(PKG_INSTALL_DIR)"

define Build/Compile
	+$(MAKE) $(PKG_JOBS) $(OPENSWAN_MAKEFLAGS) programs
ifeq ($(CONFIG_PACKAGE_kmod-crypto-ocf),y)
	+$(MAKE) $(PKG_JOBS) $(OPENSWAN_MAKEFLAGS) module
endif
endef

define Build/Install
	$(MAKE) $(OPENSWAN_MAKEFLAGS) install
endef

define Package/openswan/conffiles
/etc/ipsec.conf
/etc/ipsec.secrets
/etc/config/ipsec
endef

define Package/openswan/install
	$(CP) $(PKG_INSTALL_DIR)/* $(1)
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_DIR) $(1)/etc/hotplug.d/iface
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(CP) ./files/ipsec.init $(1)/etc/init.d/ipsec
	$(CP) ./files/ipsec.hotplug $(1)/etc/hotplug.d/iface/80-ipsec
	$(CP) ./files/ipsec.default $(1)/etc/uci-defaults/ipsec
	$(CP) ./files/*.conf $(1)/etc/
	$(CP) ./files/ipsec.secrets $(1)/etc/ipsec.secrets
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/ipsec.config $(1)/etc/config/ipsec
	rm -rf $(1)/usr/share
	rm -rf $(1)/usr/man
	rm -rf $(1)/var
	rm -rf $(1)/etc/rc.d
	find $(1) -name \*.old | xargs rm -rf
	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	$(INSTALL_DATA) files/ipsec.upgrade $(1)/lib/upgrade/keep.d/ipsec
endef

$(eval $(call BuildPackage,openswan))
$(eval $(call KernelPackage,openswan))
