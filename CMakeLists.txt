cmake_minimum_required(VERSION 3.10)
project(QmlProject LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
# 关键：启用生成 QMLLS INI 的选项
set(QT_QML_GENERATE_QMLLS_INI ON)
# 如果需要 QML types 文件，也可以:
# set(QT_QML_GENERATE_QMLTYPES ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_PREFIX_PATH "C:\\Qt\\6.9.0\\msvc2022_64\\lib\\cmake")
# 添加宏定义 utf8支持
ADD_DEFINITIONS(-DUNICODE -D_UNICODE)
set(OpenCV_DIR "E:\\code\\opencv4.9\\opencv\\build\\x64\\vc16\\lib")


set(QWindowKit_DIR ${PROJECT_SOURCE_DIR}/3rdparty/qwindowkit/lib/cmake/QWindowKit)
find_package(QWindowKit REQUIRED)


find_package(OpenCV REQUIRED COMPONENTS opencv_world)
# 查找Qt6库
find_package(Qt6 REQUIRED COMPONENTS Core Gui Quick Qml OpenGL)
if(WIN32)
    if(MSVC)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /MP")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
    elseif(MINGW)
        set(CMAKE_MAKE_PROGRAM "${CMAKE_MAKE_PROGRAM} -j${N}")
    endif()
elseif(UNIX)
    add_compile_options(-j${N})
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    # Windows 系统
    # 在这里添加针对 Windows 的设置和配置
    # 设置编译选项，包含调试符号并禁用优化

    add_link_options(/DEBUG)
    add_compile_options(/Zi /Od /RTC1)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP /utf-8")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /DEBUG:FULL /OPT:REF /OPT:ICF")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG:FULL /OPT:REF /OPT:ICF")
    #无控制台
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
    #set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:CONSOLE")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # Linux 系统
    # 在这里添加针对 Linux 的设置和配置
    # 设置编译选项，包含调试符号并禁用优化
    set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -g -O2 -Wall")
else()
    # 其他系统
    # 在这里添加针对其他系统的设置和配置
endif()
# 设置头文件目录
include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_SOURCE_DIR}/3rdparty)

# 执行程序生成路径
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE})

# 查找所有源代码文件
FILE(GLOB M_HEADER ${PROJECT_SOURCE_DIR}/include/*.h)
FILE(GLOB M_SOURCES ${PROJECT_SOURCE_DIR}/src/*.cpp)
FILE(GLOB M_UI ${PROJECT_SOURCE_DIR}/ui/*.ui)
FILE(GLOB M_QRC ${PROJECT_SOURCE_DIR}/qml/*.qrc)
FILE(GLOB M_TITLE_QRC ${PROJECT_SOURCE_DIR}/qml/resources/*.qrc)
message(STATUS "M_QRC: ${M_QRC}")
link_directories(${PROJECT_SOURCE_DIR}/lib/Release)
link_directories(${PROJECT_SOURCE_DIR}/lib/${CMAKE_BUILD_TYPE})
# 添加所有资源文件和源文件
qt_add_executable(${PROJECT_NAME} 
    ${M_HEADER} 
    ${M_SOURCES} 
    ${M_QRC}
    ${M_TITLE_QRC}
)


target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Core Qt6::Quick Qt6::Qml Qt6::OpenGL Qt6::Gui
    setupapi.lib
    legacy_stdio_definitions.lib
    QWindowKit::Quick
    ${OpenCV_LIBS})

# 如果使用Qt6, 调用qt_finalize_executable
if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(${PROJECT_NAME})
endif()

