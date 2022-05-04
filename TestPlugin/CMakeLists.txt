# Example Console App CMakeLists.txt

# To get started on a new console app, copy this entire folder (containing this file and C++
# sources) to a convenient location, and then start making modifications. For other examples of
# CMakeLists for console apps, check `extras/BinaryBuilder` and `extras/UnitTestRunner` in the JUCE
# repo.

# The first line of any CMake project should be a call to `cmake_minimum_required`, which checks
# that the installed CMake will be able to understand the following CMakeLists, and ensures that
# CMake's behaviour is compatible with the named version. This is a standard CMake command, so more
# information can be found in the CMake docs.

cmake_minimum_required(VERSION 3.12)

# The top-level CMakeLists.txt file for a project must contain a literal, direct call to the
# `project()` command. `project()` sets up some helpful variables that describe source/binary
# directories, and the current project version. This is a standard CMake command.

project(LEAF VERSION 1.0.1)

# If you've installed JUCE somehow (via a package manager, or directly using the CMake install
# target), you'll need to tell this project that it depends on the installed copy of JUCE. If you've
# included JUCE directly in your source tree (perhaps as a submodule), you'll need to tell CMake to
# include that subdirectory as part of the build.

# find_package(JUCE CONFIG REQUIRED)        # If you've installed JUCE to your system
# or
add_subdirectory(../../../mirax/sdk/juce JUCE)     # If you've put JUCE in a subdirectory called JUCE

# `juce_add_console_app` adds an executable target with the name passed as the first argument
# (ConsoleAppExample here). This target is a normal CMake target, but has a lot of extra properties
# set up by default. This function accepts many optional arguments. Check the readme at
# `docs/CMake API.md` in the JUCE repo for the full list.

juce_add_plugin(LEAF
    #ICON_BIG icon_128.png
    #ICON_SMALL icon_128.png
    IS_SYNTH TRUE                              # Is this a synth or an effect?
    PLUGIN_MANUFACTURER_CODE Mira               # A four-character manufacturer id with at least one upper-case character
    PLUGIN_CODE leaf                            # A unique four-character plugin id with at least one upper-case character
    FORMATS AU VST3 Standalone              # The formats to build. Other valid formats are: AAX Unity VST AU AUv3
    PRODUCT_NAME "LEAF")     # The name of the final executable, which can differ from the target name

# `juce_generate_juce_header` will create a JuceHeader.h for a given target, which will be generated
# into the build tree. This header should be included with `#include <JuceHeader.h>`. The include
# path for this header will be automatically added to the target. The main function of the
# JuceHeader is to include all the JUCE module headers for a particular target; if you're happy to
# include module headers directly, you probably don't need to call this.

juce_generate_juce_header(LEAF)

# `target_sources` adds source files to a target. We pass the target that needs the sources as the
# first argument, then a visibility parameter for the sources (PRIVATE is normally best practice,
# although it doesn't really affect executable targets). Finally, we supply a list of source files
# that will be built into the target. This is a standard CMake command.

file (GLOB SRC_FILES ../leaf/Src/*.* ../leaf/Externals/*.*)
target_sources(LEAF PRIVATE
    ${SRC_FILES}
    Source/LEAFLink.cpp
    Source/MyTest.cpp
    Source/PluginEditor.cpp
    Source/PluginProcessor.cpp
    Source/UIComponent.cpp
    Source/Yin.cpp
)

target_include_directories(LEAF PRIVATE
    sdk/
)

# `target_compile_definitions` adds some preprocessor definitions to our target. In a Projucer
# project, these might be passed in the 'Preprocessor Definitions' field. JUCE modules also make use
# of compile definitions to switch certain features on/off, so if there's a particular feature you
# need that's not on by default, check the module header for the correct flag to set here. These
# definitions will be visible both to your code, and also the JUCE module code, so for new
# definitions, pick unique names that are unlikely to collide! This is a standard CMake command.

target_compile_definitions(LEAF PRIVATE
    # JUCE_WEB_BROWSER and JUCE_USE_CURL would be on by default, but you might not need them.
    JUCE_USE_MP3AUDIOFORMAT=1
    JUCE_USE_WINDOWS_MEDIA_FORMAT=1
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_WEB_BROWSER=0  # If you remove this, add `NEEDS_WEB_BROWSER TRUE` to the `juce_add_console_app` call
    JUCE_USE_CURL=0)    # If you remove this, add `NEEDS_CURL TRUE` to the `juce_add_console_app` call

# If the target needs extra binary assets, they can be added here. The first argument is the name of
# a new static library target that will include all the binary resources. There is an optional
# `NAMESPACE` argument that can specify the namespace of the generated binary data class. Finally,
# the SOURCES argument should be followed by a list of source files that should be built into the
# static library. These source files can be of any kind (wav data, images, fonts, icons etc.).
# Conversion to binary-data will happen when the target is built.

#file (GLOB SOUND_FILES ../../media/*.wav)
#juce_add_binary_data(BinaryData SOURCES ${SOUND_FILES})

# `target_link_libraries` links libraries and JUCE modules to other libraries or executables. Here,
# we're linking our executable target to the `juce::juce_core` module. Inter-module dependencies are
# resolved automatically. If you'd generated a binary data target above, you would need to link to
# it here too. This is a standard CMake command.

target_link_libraries(LEAF PRIVATE
    #BinaryData            # If you'd created a binary data target, you'd link to it here
    juce::juce_audio_formats
    juce::juce_cryptography
    juce::juce_gui_extra
    juce::juce_audio_utils)
