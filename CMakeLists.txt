#
# Copyright 2019, Data61, CSIRO (ABN 41 687 119 230)
#
# SPDX-License-Identifier: BSD-2-Clause
#

cmake_minimum_required(VERSION 3.7.2)

include(settings.cmake)

project(sel4test C CXX ASM)

find_package(seL4 REQUIRED)
find_package(elfloader-tool REQUIRED)

# Set the roottask CNode size to be large enough for DTB, timer caps, etc
# Note that this works for the platforms that we have at the moment, and may
# need to be increased in the future
set(KernelRootCNodeSizeBits 13 CACHE INTERNAL "")

# Set our custom domain schedule
set(KernelDomainSchedule "${CMAKE_CURRENT_LIST_DIR}/domain_schedule.c" CACHE INTERNAL "")
sel4_import_kernel()

if((NOT Sel4testAllowSettingsOverride) AND (KernelArchARM OR KernelArchRiscV))
    # Elfloader settings that correspond to how Data61 sets its boards up.
    ApplyData61ElfLoaderSettings(${KernelPlatform} ${KernelSel4Arch})
endif()
elfloader_import_project()

add_subdirectory(apps/sel4test-driver)

include(simulation)
if(KernelSel4ArchX86_64)
    SetSimulationScriptProperty(MEM_SIZE "3G")
endif()
if(KernelPlatformQEMUArmVirt)
    SetSimulationScriptProperty(MEM_SIZE "2G")
endif()

GenerateSimulateScript()

# Generate HSS payload binary. Only seL4 image is included.
if(PolarfireAmp)
    find_package(gen_hss_payload REQUIRED)
    gen_hss_payload_import_project()
endif()
