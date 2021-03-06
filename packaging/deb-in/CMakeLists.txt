# Copyright (c) 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

IF(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
  RETURN()
ENDIF()

IF (NOT DEFINED DEB_CODENAME)
  execute_process(
    COMMAND lsb_release -cs
    OUTPUT_VARIABLE DEB_CODENAME
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  SET (DEB_CODENAME ${DEB_CODENAME} CACHE STRING "")
ENDIF()
# Commercial or community
IF (DEB_PRODUCT STREQUAL "commercial")
  SET (DEB_COPYRIGHT_UPSTREAMNAME "MySQL Commercial Server ${MYSQL_BASE_VERSION}")
  SET (DEB_PRODUCTNAME "commercial")
  SET (DEB_PRODUCTNAMEC "Commercial")
  SET (DEB_NOTPRODUCTNAME "community")
  SET (DEB_LICENSENAME "Advanced")
  SET (DEB_INSTALL_LICENSEFILE "LICENSE.mysql")
  SET (DEB_SERVERVERSION "${VERSION}+commercial-1")
  # List of plugins that are only in commercial packages
  # Plugins that are in both community and commercial should NOT be added here
  SET (DEB_INCLUDE_BINARIES_EXTRA
"
debian/extra/audit_log-plugin
debian/extra/authentication_pam-plugin
debian/extra/firewall-plugin
debian/extra/keyring_okv-plugin
debian/extra/openssl_udf-plugin
debian/extra/thread_pool-plugin
")
  SET (DEB_INSTALL_SERVER_PLUGINS
"
usr/lib/mysql/plugin/audit_log.so
usr/lib/mysql/plugin/authentication_pam.so
usr/lib/mysql/plugin/keyring_okv.so
usr/lib/mysql/plugin/openssl_udf.so
usr/lib/mysql/plugin/thread_pool.so
usr/lib/mysql/plugin/firewall.so
usr/lib/mysql/plugin/debug/audit_log.so
usr/lib/mysql/plugin/debug/authentication_pam.so
usr/lib/mysql/plugin/debug/keyring_okv.so
usr/lib/mysql/plugin/debug/openssl_udf.so
usr/lib/mysql/plugin/debug/thread_pool.so
usr/lib/mysql/plugin/debug/firewall.so
")
  SET (DEB_REMOVEPATTERN "gpl.in")
ELSE()
  SET (DEB_PRODUCT "community")
  SET (DEB_COPYRIGHT_UPSTREAMNAME "MySQL Server ${MYSQL_BASE_VERSION}")
  SET (DEB_PRODUCTNAME "community")
  SET (DEB_PRODUCTNAMEC "Community")
  SET (DEB_NOTPRODUCTNAME "commercial")
  SET (DEB_LICENSENAME "GPL")
  SET (DEB_INSTALL_LICENSEFILE "COPYING")
  SET (DEB_SERVERVERSION "${VERSION}-1")
  SET (DEB_REMOVEPATTERN "com.in")
ENDIF()

# Platform specifics. The differences are generally only distro version
# and whether or not systemd and/or apparmor are available
IF (DEB_CODENAME STREQUAL "wheezy")
  SET (DEB_PLATFORMRELEASE "debian7")
  SET (DEB_CONTROL_BDEPS "")
  SET (DEB_INSTALL_SOURCE_XZ "")
  SET (DEB_RULES_INSTALL_SYSTEMD "")
  SET (DEB_RULES_INSTALL_APPARMOR "")
  SET (DEB_RULES_APPARMOR_LOAD "")
  SET (DEB_RULES_SYSTEMD_ENABLE "")
  SET (DEB_RULES_SYSTEMD_START "")
  SET (DEB_INSTALL_SERVER_SYSTEMD "")
  SET (DEB_INSTALL_SERVER_APPARMOR "")
  SET (DEB_SERVICE_SERVER_EXECPRE "")
  SET (DEB_INIT_APPARMOR "")
ELSEIF(DEB_CODENAME STREQUAL "jessie")
  SET (DEB_PLATFORMRELEASE "debian8")
  SET (DEB_CONTROL_BDEPS "dh-systemd")
  SET (DEB_INSTALL_SOURCE_XZ "../*.tar.xz usr/src/mysql/")
  SET (DEB_RULES_INSTALL_SYSTEMD
    "install -m 0755 debian/extra/mysql-systemd-start debian/tmp/usr/share/mysql/")
  SET (DEB_RULES_INSTALL_APPARMOR "")
  SET (DEB_RULES_APPARMOR_LOAD "")
  SET (DEB_RULES_SYSTEMD_ENABLE "dh_systemd_enable --name=mysql")
  SET (DEB_RULES_SYSTEMD_START "dh_systemd_start --restart-after-upgrade")
  SET (DEB_INSTALL_SERVER_SYSTEMD "usr/share/mysql/mysql-systemd-start")
  SET (DEB_INSTALL_SERVER_APPARMOR "")
  SET (DEB_SERVICE_SERVER_EXECPRE
    "ExecStartPre=/usr/share/mysql/mysql-systemd-start pre")
  SET (DEB_INIT_APPARMOR "")
ELSEIF(DEB_CODENAME STREQUAL "precise")
  SET (DEB_PLATFORMRELEASE "ubuntu12.04")
  SET (DEB_CONTROL_BDEPS "dh-apparmor")
  SET (DEB_INSTALL_SOURCE_XZ "")
  SET (DEB_RULES_INSTALL_SYSTEMD "")
  SET (DEB_RULES_INSTALL_APPARMOR
    "install -g root -o root -m 0644 -D debian/extra/apparmor-profile debian/tmp/etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_RULES_APPARMOR_LOAD
    "dh_apparmor -pmysql-${DEB_PRODUCTNAME}-server --profile-name=usr.sbin.mysqld")
  SET (DEB_RULES_SYSTEMD_ENABLE "")
  SET (DEB_RULES_SYSTEMD_START "")
  SET (DEB_INSTALL_SERVER_SYSTEMD "")
  SET (DEB_INSTALL_SERVER_APPARMOR "etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_SERVICE_SERVER_EXECPRE "")
  SET (DEB_INIT_APPARMOR "/lib/init/apparmor-profile-load usr.sbin.mysqld")
ELSEIF(DEB_CODENAME STREQUAL "trusty")
  SET (DEB_PLATFORMRELEASE "ubuntu14.04")
  SET (DEB_CONTROL_BDEPS "dh-apparmor")
  SET (DEB_INSTALL_SOURCE_XZ "")
  SET (DEB_RULES_INSTALL_SYSTEMD "")
  SET (DEB_RULES_INSTALL_APPARMOR
    "install -g root -o root -m 0644 -D debian/extra/apparmor-profile debian/tmp/etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_RULES_APPARMOR_LOAD
    "dh_apparmor -pmysql-${DEB_PRODUCTNAME}-server --profile-name=usr.sbin.mysqld")
  SET (DEB_RULES_SYSTEMD_ENABLE "")
  SET (DEB_RULES_SYSTEMD_START "")
  SET (DEB_INSTALL_SERVER_SYSTEMD "")
  SET (DEB_INSTALL_SERVER_APPARMOR "etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_SERVICE_SERVER_EXECPRE "")
  SET (DEB_INIT_APPARMOR "/lib/init/apparmor-profile-load usr.sbin.mysqld")
ELSEIF(DEB_CODENAME STREQUAL "wily")
  SET (DEB_PLATFORMRELEASE "ubuntu15.10")
  SET (DEB_CONTROL_BDEPS "dh-apparmor, dh-systemd (>=1.5)")
  SET (DEB_INSTALL_SOURCE_XZ "../*.tar.xz usr/src/mysql/")
  SET (DEB_RULES_INSTALL_SYSTEMD
    "install -m 0755 debian/extra/mysql-systemd-start debian/tmp/usr/share/mysql/")
  SET (DEB_RULES_INSTALL_APPARMOR
    "install -g root -o root -m 0644 -D debian/extra/apparmor-profile debian/tmp/etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_RULES_APPARMOR_LOAD
    "dh_apparmor -pmysql-community-server --profile-name=usr.sbin.mysqld")
  SET (DEB_RULES_SYSTEMD_ENABLE "dh_systemd_enable --name=mysql")
  SET (DEB_RULES_SYSTEMD_START "dh_systemd_start --restart-after-upgrade")
  SET (DEB_INSTALL_SERVER_SYSTEMD "usr/share/mysql/mysql-systemd-start")
  SET (DEB_INSTALL_SERVER_APPARMOR "etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_SERVICE_SERVER_EXECPRE
    "ExecStartPre=/usr/share/mysql/mysql-systemd-start pre")
  SET (DEB_INIT_APPARMOR "/lib/init/apparmor-profile-load usr.sbin.mysqld")
ELSEIF(DEB_CODENAME STREQUAL "xenial")
  SET (DEB_PLATFORMRELEASE "ubuntu16.04")
  SET (DEB_CONTROL_BDEPS "dh-apparmor, dh-systemd (>=1.5)")
  SET (DEB_INSTALL_SOURCE_XZ "../*.tar.xz usr/src/mysql/")
  SET (DEB_RULES_INSTALL_SYSTEMD
    "install -m 0755 debian/extra/mysql-systemd-start debian/tmp/usr/share/mysql/")
  SET (DEB_RULES_INSTALL_APPARMOR
    "install -g root -o root -m 0644 -D debian/extra/apparmor-profile debian/tmp/etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_RULES_APPARMOR_LOAD
    "dh_apparmor -pmysql-${DEB_PRODUCTNAME}-server --profile-name=usr.sbin.mysqld")
  SET (DEB_RULES_SYSTEMD_ENABLE "dh_systemd_enable --name=mysql")
  SET (DEB_RULES_SYSTEMD_START "dh_systemd_start --restart-after-upgrade")
  SET (DEB_INSTALL_SERVER_SYSTEMD "usr/share/mysql/mysql-systemd-start")
  SET (DEB_INSTALL_SERVER_APPARMOR "etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_SERVICE_SERVER_EXECPRE
    "ExecStartPre=/usr/share/mysql/mysql-systemd-start pre")
  SET (DEB_INIT_APPARMOR "/lib/apparmor/profile-load usr.sbin.mysqld")
ELSEIF(DEB_CODENAME STREQUAL "yakkety")
  SET (DEB_PLATFORMRELEASE "ubuntu16.10")
  SET (DEB_CONTROL_BDEPS "dh-apparmor, dh-systemd (>=1.5)")
  SET (DEB_INSTALL_SOURCE_XZ "../*.tar.xz usr/src/mysql/")
  SET (DEB_RULES_INSTALL_SYSTEMD
    "install -m 0755 debian/extra/mysql-systemd-start debian/tmp/usr/share/mysql/")
  SET (DEB_RULES_INSTALL_APPARMOR
    "install -g root -o root -m 0644 -D debian/extra/apparmor-profile debian/tmp/etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_RULES_APPARMOR_LOAD
    "dh_apparmor -pmysql-${DEB_PRODUCTNAME}-server --profile-name=usr.sbin.mysqld")
  SET (DEB_RULES_SYSTEMD_ENABLE "dh_systemd_enable --name=mysql")
  SET (DEB_RULES_SYSTEMD_START "dh_systemd_start --restart-after-upgrade")
  SET (DEB_INSTALL_SERVER_SYSTEMD "usr/share/mysql/mysql-systemd-start")
  SET (DEB_INSTALL_SERVER_APPARMOR "etc/apparmor.d/usr.sbin.mysqld")
  SET (DEB_SERVICE_SERVER_EXECPRE
    "ExecStartPre=/usr/share/mysql/mysql-systemd-start pre")
  SET (DEB_INIT_APPARMOR "/lib/apparmor/profile-load usr.sbin.mysqld")
ELSE()
  MESSAGE(STATUS
    "Skipping deb packaging on unsupported platform ${DEB_CODENAME}.")
  RETURN()
ENDIF()

# All files are configured and copied to the debian/ directory, which is used 
# by debuild to make the packages
SET (DEB_ROOT ${CMAKE_SOURCE_DIR}/packaging/deb-in)
FILE(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/debian)
FILE(GLOB_RECURSE SOURCEFILES RELATIVE "${DEB_ROOT}" "${DEB_ROOT}/*")
# Skip the copyright file from the "other" product type
LIST(REMOVE_ITEM SOURCEFILES "copyright.${DEB_REMOVEPATTERN}")
LIST(REMOVE_ITEM SOURCEFILES "CMakeLists.txt")
FOREACH (SOURCEFILE ${SOURCEFILES})
  STRING(REGEX REPLACE "-packagesource-" "-${DEB_PRODUCTNAME}-"
    TMPFILE ${SOURCEFILE})
  # Strip away the input file endings from the destination filenames
  STRING(REGEX REPLACE ".in$" "" TMPFILE ${TMPFILE})
  STRING(REGEX REPLACE ".gpl$" "" TMPFILE ${TMPFILE})
  STRING(REGEX REPLACE ".com$" "" DESTFILE ${TMPFILE})
  CONFIGURE_FILE("${DEB_ROOT}/${SOURCEFILE}"
    "${CMAKE_BINARY_DIR}/debian/${DESTFILE}" @ONLY)
ENDFOREACH()
EXECUTE_PROCESS(
  COMMAND chmod +x ${CMAKE_BINARY_DIR}/debian/rules
)

