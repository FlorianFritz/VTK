cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
foreach(p
    CMP0071 # 3.10: Let AUTOMOC and AUTOUIC process GENERATED files
    )
  if(POLICY ${p})
    cmake_policy(SET ${p} NEW)
  endif()
endforeach()

project(EasyView)

find_package(VTK REQUIRED)
vtk_module_config(VTK
  vtkCommonCore
  vtkCommonDataModel
  vtkGUISupportQt
  vtkIOInfovis
  vtkInfovisCore
  vtkViewsCore
  vtkViewsInfovis
  vtkViewsQt
)
include(${VTK_USE_FILE})

if("${VTK_QT_VERSION}" STREQUAL "")
  message(FATAL_ERROR "VTK was not built with Qt")
endif()

# Use the include path and library for Qt that is used by VTK.
include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
)

# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

# Set your files and resources here
set(Srcs main.cxx EasyView.cxx)
set(UIs EasyView.ui)
set(Hdrs EasyView.h)
set(MOC_Hdrs EasyView.h)
set(Rcs Icons/icons.qrc)

# It is nice to have the ui in the windows project file.
# Right-click, choose Open With... press the Add button,
# and then find designer.exe and set as default.
# After this just click on it to bring it up in designer.
if(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv)")
  set(Srcs ${Srcs} ${UIs})
endif(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv)")

# We have ui files, this will bring in the macro: qt5_wrap_ui
find_package(Qt5 COMPONENTS Widgets REQUIRED QUIET)
qt5_wrap_ui(UI_Srcs ${UIs})
qt5_add_resources(Rcs_Srcs ${Rcs})

source_group("Resources" FILES
  ${UIs}
  ${Rcs}
  ${EXE_ICON} # Not present
)

source_group("Generated" FILES
  ${UI_Srcs}
  ${MOC_Srcs}
  ${Rcs_Srcs}
)

#  set_source_files_PROPERTIES(${Srcs} PROPERTIES
#                              OBJECT_DEPENDS "${UI_Srcs}")

add_executable(EasyView
  MACOSX_BUNDLE  ${Srcs} ${Hdrs} ${UI_Srcs} ${MOC_Hdrs} ${Rcs_Srcs})

qt5_use_modules(EasyView Core Gui Widgets)

target_link_libraries(EasyView ${VTK_LIBRARIES})
