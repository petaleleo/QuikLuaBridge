# QuikLuaBridge
Инструкция по настройке и компиляции ZeroMQ, Lua, Protobuf, Python, Quik

Основано на https://github.com/Enfernuz/quik-lua-rpc/
Компилируем/собираем всё сами.

Содержание
=================

  * [Собираем LUA](#LUA)
  * [Как это работает?](#Как-это-работает)
  
LUA
--------

1. Нужен VisualStudio. Берём тут Community Version: https://visualstudio.microsoft.com/ru/
2. Берём актуальную версию LUA для QUIK отсюда https://www.lua.org/ftp/#source
3. Запускаем необходимо окружение компилятора из установленного VisualStudio: x64 Native Tools Command Prompt for VS 2022
4. Собираем Lua:

```bat
call cl /MD /O2 /c /DLUA_BUILD_AS_DLL src/*.c
ren lua.obj lua.o
ren luac.obj luac.o
link /DLL /IMPLIB:lua54.lib /OUT:lua54.dll *.obj
link /OUT:lua54.exe lua.o lua54.lib
lib /OUT:lua54-static.lib *.obj
link /OUT:luac.exe luac.o lua54-static.lib
del *.obj
del *.exp
del *.o
```
