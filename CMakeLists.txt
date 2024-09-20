cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

# Поиск библиотеки Boost
find_package(Boost COMPONENTS unit_test_framework REQUIRED)

# Конфигурация файла версии
configure_file(version.h.in version.h)

# Определение исполняемых файлов и библиотек
add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

# Установка стандартов C++
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Установка директории для заголовков
target_include_directories(helloworld PRIVATE "${CMAKE_BINARY_DIR}")

# Настройка тестового исполняемого файла
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

# Связывание библиотек
target_link_libraries(helloworld_cli PRIVATE helloworld)
target_link_libraries(test_version ${Boost_LIBRARIES} helloworld)

# Настройка параметров компиляции в зависимости от компилятора
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else ()
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

# Установка целевых файлов для установки
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Установка генератора CPack в зависимости от платформы
if (APPLE)
    set(CPACK_GENERATOR "Bundle")  # Для создания .app или .pkg на macOS
else()
    set(CPACK_GENERATOR "DEB")  # Для создания .deb на Linux
endif()

# Установка имени пакета для CPack
set(CPACK_PACKAGE_NAME "LABABAB")  # Замените на имя вашего приложения

# Установка версии пакета для CPack
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_CONTACT "example@example.com")

# Дополнительные метаданные для macOS (если необходимо)
if (APPLE)
    set(CPACK_BUNDLE_NAME "${CPACK_PACKAGE_NAME}")
    set(CPACK_BUNDLE_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/resources/Info.plist")  # Укажите правильный путь к вашему Info.plist
    set(CPACK_BUNDLE_ICON "${CMAKE_CURRENT_SOURCE_DIR}/resources/ic.icns")  # Укажите правильный путь к вашему файлу иконки
endif()

include(CPack)  # Включение CPack

enable_testing()  # Включение тестирования

add_test(test_version test_version)  # Добавление теста для запуска через CTest