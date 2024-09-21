cmake_minimum_required(VERSION 3.10)  # Минимальная версия CMake
project(HelloWorld)                     # Имя проекта

set(CMAKE_CXX_STANDARD 11)              # Установка стандарта C++
set(CMAKE_CXX_STANDARD_REQUIRED True)   # Обязательное использование стандарта

add_executable(hello hello.cpp)         # Создание исполняемого файла