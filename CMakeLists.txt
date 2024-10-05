cmake_minimum_required(VERSION 3.10) # Устанавливает минимальную версию CMake, необходимую для сборки проекта.

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version") # Определяет версию патча как внутреннюю переменную.
set(PROJECT_VERSION 0.0.${PATCH_VERSION}) # Формирует полную версию проекта, включая номер патча.

project(helloworld VERSION ${PROJECT_VERSION}) # Определяет название проекта и его версию.

find_package(Boost COMPONENTS unit_test_framework REQUIRED) # Находит библиотеку Boost, необходимую для тестирования. Если не найдена, выдаст ошибку.

configure_file(version.h.in version.h) # Настраивает файл version.h на основе шаблона version.h.in.

add_executable(helloworld_cli main.cpp) # Создает исполняемый файл helloworld_cli из исходного файла main.cpp.
add_library(helloworld lib.cpp) # Создает библиотеку helloworld из исходного файла lib.cpp.
add_executable(test_version test_version.cpp) # Создает исполняемый файл test_version из test_version.cpp для запуска тестов.

# Устанавливает свойства для целевых объектов, чтобы использовать стандарт C++14 во время компиляции.
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14 # Устанавливает стандарт C++14.
    CXX_STANDARD_REQUIRED ON # Обеспечивает ошибку, если стандарт C++14 недоступен.
)

# Определяет включаемые каталоги для библиотеки helloworld, делая двоичный каталог доступным только для нее.
target_include_directories(helloworld PRIVATE "${CMAKE_BINARY_DIR}")

# Устанавливает свойства для test_version, определяя макрос BOOST_TEST_DYN_LINK и включая каталог Boost.
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK # Определяет макрос для динамической линковки Boost.
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR} # Включает директорию с заголовочными файлами Boost.
)

target_link_libraries(helloworld_cli PRIVATE helloworld) # Связывает библиотеку helloworld с исполняемым файлом helloworld_cli.
target_link_libraries(test_version ${Boost_LIBRARIES} helloworld) # Связывает библиотеки Boost и helloworld с исполняемым файлом test_version.

# Устанавливает уровни предупреждений компилятора в зависимости от используемого компилятора.
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4) # Устанавливает уровень предупреждений 4 для MSVC.
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else ()
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror) # Устанавливает строгие предупреждения для GCC/Clang.
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

install(TARGETS helloworld_cli RUNTIME DESTINATION bin) # Определяет правила установки для helloworld_cli в каталог bin.

# Установка генератора CPack в зависимости от платформы.
if (APPLE)
    set(CPACK_GENERATOR "Bundle")  # Для создания пакетов на macOS.
else()
    set(CPACK_GENERATOR "DEB")  # Для создания .deb на Linux.
endif()

set(CPACK_BUNDLE_NAME "LABABAB")  # Задает имя пакета для CPack.

set(CPACK_BUNDLE_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/resources/Info.plist")  # Указывает путь к Info.plist для macOS.

set(CPACK_BUNDLE_ICON "${CMAKE_CURRENT_SOURCE_DIR}/resources/ic.icns")  # Указывает путь к файлу иконки.

set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}") # Устанавливает версию пакета для CPack.
set(CPACK_PACKAGE_CONTACT "example@example.com") # Указывает контактную информацию для пакета.

include(CPack) # Включает поддержку CPack для упаковки проекта.

enable_testing() # Включает поддержку тестирования в проекте, позволяя определять и запускать тесты.

add_test(test_version test_version) # Добавляет тест test_version в систему тестирования CMake.