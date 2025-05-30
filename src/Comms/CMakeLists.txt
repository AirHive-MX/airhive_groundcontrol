add_subdirectory(AirLink)
add_subdirectory(MockLink)

find_package(Qt6 REQUIRED COMPONENTS Core Network Qml)

qt_add_library(Comms STATIC
    LinkConfiguration.cc
    LinkConfiguration.h
    LinkInterface.cc
    LinkInterface.h
    LinkManager.cc
    LinkManager.h
    LogReplayLink.cc
    LogReplayLink.h
    LogReplayLinkController.cc
    LogReplayLinkController.h
    MAVLinkProtocol.cc
    MAVLinkProtocol.h
    TCPLink.cc
    TCPLink.h
    UDPLink.cc
    UDPLink.h
)

target_link_libraries(Comms
    PRIVATE
        Qt6::Qml
        MockLink
        PositionManager
        QGC
        QmlControls
        Settings
        Utilities
        Vehicle
    PUBLIC
        Qt6::Core
        Qt6::Network
        AirLink
        MAVLink
)

target_include_directories(Comms PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

#===========================================================================#

if(QGC_NO_SERIAL_LINK)
    target_compile_definitions(Comms PUBLIC QGC_NO_SERIAL_LINK)
else()
    target_sources(Comms
        PRIVATE
            QGCSerialPortInfo.cc
            QGCSerialPortInfo.h
            SerialLink.cc
            SerialLink.h
            UdpIODevice.cc
            UdpIODevice.h
    )

    target_link_libraries(Comms
        PRIVATE
            GPS
            PositionManager
    )

    if(ANDROID)
        add_subdirectory(${CMAKE_SOURCE_DIR}/android/libs/qtandroidserialport qtandroidserialport.build)
        target_sources(qtandroidserialport
            PRIVATE
                ${CMAKE_SOURCE_DIR}/android/src/AndroidSerial.cc
                ${CMAKE_SOURCE_DIR}/android/src/AndroidSerial.h
        )
        target_include_directories(qtandroidserialport PUBLIC ${CMAKE_SOURCE_DIR}/android/src)
        target_link_libraries(Comms PUBLIC qtandroidserialport)
    else()
        find_package(Qt6 REQUIRED COMPONENTS SerialPort)
        target_link_libraries(Comms PUBLIC Qt6::SerialPort)
    endif()
endif()

#===========================================================================#

if(QGC_ENABLE_BLUETOOTH)
    find_package(Qt6 REQUIRED COMPONENTS Bluetooth)
    target_link_libraries(Comms PUBLIC Qt6::Bluetooth)
    target_sources(Comms
        PRIVATE
            BluetoothLink.cc
            BluetoothLink.h
    )
    target_compile_definitions(Comms PUBLIC QGC_ENABLE_BLUETOOTH)
endif()

#===========================================================================#

if(QGC_ZEROCONF_ENABLED)
    message(STATUS "Building QMDNSEngine")

    include(CPM)
    CPMAddPackage(
        NAME qmdnsengine
        GITHUB_REPOSITORY nitroshare/qmdnsengine
        GIT_TAG master
        OPTIONS
            "BUILD_DOC OFF"
            "BUILD_EXAMPLES OFF"
            "BUILD_TESTS OFF"
    )

    if(TARGET qmdnsengine)
        target_link_libraries(Comms PUBLIC qmdnsengine)
        target_compile_definitions(Comms PUBLIC QGC_ZEROCONF_ENABLED)
    endif()
endif()

# target_compile_definitions(Comms PUBLIC QABSTRACTSOCKET_DEBUG)
