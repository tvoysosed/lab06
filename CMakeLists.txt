cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)

project(print)

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)
if(BUILD_TESTS) # если тесты построения пройдены
enable_testing() # вызываем функцию, включающую тесты
file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp) # создаем список файлов, которые находятся в директории tests и имеют расширение .cpp, и записываем его в переменную ${PROJECT_NAME}_TEST_SOURCES
add_executable(check ${${PROJECT_NAME}_TEST_SOURCES}) # добавляем в проект исполняемый файл с именем check (исполняемый проект будет построен из файлов, хранящихся в ${PROJECT_NAME}_TEST_SOURCES )
target_link_libraries(check ${PROJECT_NAME} ${DEPENDS_LIBRARIES}) # добавляем к бинарному файлу подключаемые библиотеки
add_test(NAME check COMMAND check "-s" "-r" "compact" "--use-colour" "yes") # добавляем тест check. Указываем тестовую командную строку: check "-s" "-r" "compact" "--use-colour" "yes". 
endif() # конец условного оператора
