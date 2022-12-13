# QuikLuaBridge
Инструкция по настройке и компиляции ZeroMQ, Lua, Protobuf, Python, Quik

Основано на https://github.com/Enfernuz/quik-lua-rpc

Компилируем и собираем всё сами.

Содержание
=================

  * [Собираем LUA](#LUA)
  * [Собираем LIBZMQ](#LIBZMQ)
  * [Собираем PB.DLL](#LUA-PROTOBUF)
  
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

LIBZMQ
--------

LUA-PROTOBUF
--------
Для работы и компиляции protobuf в LUA нужно получить pb.dll: 
1. Проект берём тут: https://github.com/starwing/lua-protobuf
2. Т.к. LUA в QUIK работает в двух потоках, а библиотека не учитывает это, то необходимо внести изменение в код в модуле pb.c:

```C
#include <windows.h>
static volatile int globCount=0;


static int Lpb_encode(lua_State *L) {
	
    lpb_State *LS = lpb_lstate(L);
	
	while (globCount>0) Sleep(10);
	globCount++;

    const pb_Type *t = lpb_type(LS, lpb_checkslice(L, 1));
    lpb_Env e;
    argcheck(L, t!=NULL, 1, "type '%s' does not exists", lua_tostring(L, 1));
    luaL_checktype(L, 2, LUA_TTABLE);
    e.L = L, e.LS = LS, e.b = test_buffer(L, 3);
    if (e.b == NULL) pb_resetbuffer(e.b = &LS->buffer);
    lua_pushvalue(L, 2);
    if (e.LS->use_enc_hooks) lpb_useenchooks(L, e.LS, t);
    lpbE_encode(&e, t, -1);
    if (e.b != &LS->buffer)
        lua_settop(L, 3);
    else {
        lua_pushlstring(L, pb_buffer(e.b), pb_bufflen(e.b));
        pb_resetbuffer(e.b);
    }
	
	
	globCount--;
		
    return 1;
}
```


3. Компилируем: 
```bat
cl /O2 /LD /Fpb.dll /DLUA_BUILD_AS_DLL /I "...\lua-5.4.1\src" pb.c log.c "....\lua-5.4.1\lua54.lib"
```

5. Полученный файл нужно скопировать в папку QUIK
