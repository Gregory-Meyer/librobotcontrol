############################################################################
#
#   Copyright (c) 2018 PX4 Development Team. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in
#    the documentation and/or other materials provided with the
#    distribution.
# 3. Neither the name PX4 nor the names of its contributors may be
#    used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
# OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
# AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
# ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#
############################################################################

cmake_minimum_required(VERSION 3.8)

project(RobotControl VERSION 1.0.4 LANGUAGES C)

find_package(Threads REQUIRED)

add_library(robotcontrol STATIC
	src/bmp/bmp.c
	src/io/adc.c
	src/io/encoder_eqep.c
	src/io/gpio.c
	src/io/i2c.c
	src/io/pwm.c
	src/io/spi.c
	src/io/uart.c
	src/math/algebra.c
	src/math/algebra_common.c
	src/math/filter.c
	src/math/kalman.c
	src/math/matrix.c
	src/math/other.c
	src/math/polynomial.c
	src/math/quaternion.c
	src/math/ring_buffer.c
	src/math/vector.c
	src/mpu/mpu.c
	src/pru/encoder_pru.c
	src/pru/pru.c
	src/pru/servo.c
	src/button.c
	src/cpu.c
	src/deprecated.c
	src/dsm.c
	src/encoder.c
	src/led.c
	src/mavlink_udp.c
	src/model.c
	src/motor.c
	src/pinmux.c
	src/pthread.c
	src/start_stop.c
	src/time.c
	src/version.c
)
target_compile_features(robotcontrol PUBLIC c_std_99)
target_include_directories(robotcontrol
    PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src
)
target_link_libraries(robotcontrol PUBLIC m rt Threads::Threads)
add_library(RobotControl::RobotControl ALIAS robotcontrol)

add_executable(rc_calibrate_accel examples/rc_calibrate_accel.c)
target_compile_features(rc_calibrate_accel PUBLIC c_std_99)
target_link_libraries(rc_calibrate_accel RobotControl::RobotControl)

add_executable(rc_calibrate_gyro examples/rc_calibrate_gyro.c)
target_compile_features(rc_calibrate_gyro PUBLIC c_std_99)
target_link_libraries(rc_calibrate_gyro RobotControl::RobotControl)

add_executable(rc_calibrate_mag examples/rc_calibrate_mag.c)
target_compile_features(rc_calibrate_mag PUBLIC c_std_99)
target_link_libraries(rc_calibrate_mag RobotControl::RobotControl)

add_executable(rc_test_mpu examples/rc_test_mpu.c)
target_compile_features(rc_test_mpu PUBLIC c_std_99)
target_link_libraries(rc_test_mpu RobotControl::RobotControl)

##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/RobotControl)

install(TARGETS robotcontrol
    EXPORT robotcontrol-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

#This is required so that the exported target has the name RobotControl and not robotcontrol
set_target_properties(robotcontrol PROPERTIES EXPORT_NAME RobotControl)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT robotcontrol-targets
  FILE
    RobotControlTargets.cmake
  NAMESPACE
    RobotControl::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/RobotControlConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/RobotControlConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/RobotControlConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

#Install the config, configversion and custom find modules
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/RobotControlConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/RobotControlConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

export(EXPORT robotcontrol-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/RobotControlTargets.cmake NAMESPACE RobotControl::)

#Register package in user's package registry
export(PACKAGE RobotControl)
