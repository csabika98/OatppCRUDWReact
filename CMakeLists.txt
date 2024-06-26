cmake_minimum_required(VERSION 3.5)  # Updated from 3.1
project(crud)

set(CMAKE_CXX_STANDARD 17)

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_MANAGER} /GR")
endif()

if(MSVC)
    string(REGEX REPLACE "/EH[a-z]-" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
endif()


add_library(crud-lib
        src/controller/StaticController.hpp
        src/controller/UserController.hpp
        src/db/UserDb.hpp
        src/dto/PageDto.hpp
        src/dto/StatusDto.hpp
        src/dto/UserDto.hpp
        src/service/UserService.cpp
        src/service/UserService.hpp
        src/AppComponent.hpp
        src/DatabaseComponent.hpp
        src/SwaggerComponent.hpp
        src/ErrorHandler.cpp
        src/ErrorHandler.hpp
        src/service/TemplateService.hpp)

## include directories

target_include_directories(crud-lib PUBLIC src)


## link libs

find_package(oatpp          1.4.0 REQUIRED)
find_package(oatpp-swagger  1.4.0 REQUIRED)
find_package(oatpp-sqlite   1.4.0 REQUIRED)

target_link_libraries(crud-lib
        # Oat++
        PUBLIC oatpp::oatpp
        PUBLIC oatpp::oatpp-swagger
        PUBLIC oatpp::oatpp-sqlite
)

# If CMake can't find SQLite3:
#
# 1. Make sure that you've built oatpp-sqlite with -DOATPP_SQLITE_AMALGAMATION=ON flag
# 2. If you are not willing to use SQLite amalgamation then uncomment the following lines:

#find_package(SQLite3 REQUIRED)

#target_link_libraries(crud-lib
#       PUBLIC SQLite::SQLite3
#)

# FOR WINDOWS
#set(OATPP_SWAGGER_RES_PATH "C:/Program Files (x86)/oatpp-swagger/include/oatpp-1.4.0/bin/oatpp-swagger/res")
#message(STATUS "Swagger Resources Path: ${OATPP_SWAGGER_RES_PATH}")
#add_definitions(-DOATPP_SWAGGER_RES_PATH="${OATPP_SWAGGER_RES_PATH}")

add_definitions(
        ## define path to swagger-ui static resources folder
        -DOATPP_SWAGGER_RES_PATH="${oatpp-swagger_INCLUDE_DIRS}/../bin/oatpp-swagger/res"


        ## SQLite database file
        -DDATABASE_FILE="${CMAKE_CURRENT_SOURCE_DIR}/db.sqlite"
        ## SQLite database test file
        -DTESTDATABASE_FILE="${CMAKE_CURRENT_SOURCE_DIR}/test-db.sqlite"

        ## Path to database migration scripts
        -DDATABASE_MIGRATIONS="${CMAKE_CURRENT_SOURCE_DIR}/sql"
)

if(CMAKE_SYSTEM_NAME MATCHES Linux)
    find_package(Threads REQUIRED)
    target_link_libraries(crud-lib INTERFACE Threads::Threads ${CMAKE_DL_LIBS})
endif()

## add executables

add_executable(crud-exe src/App.cpp
        src/service/TemplateService.hpp)
target_link_libraries(crud-exe crud-lib)

add_executable(crud-test
        test/tests.cpp
        test/app/TestClient.hpp
        test/app/TestDatabaseComponent.hpp
        test/app/TestComponent.hpp
        test/UserControllerTest.hpp
        test/UserControllerTest.cpp
        src/service/TemplateService.hpp)
target_link_libraries(crud-test crud-lib)

enable_testing()
add_test(project-tests crud-test)
