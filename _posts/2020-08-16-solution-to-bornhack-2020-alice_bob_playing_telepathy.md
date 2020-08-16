---
layout: post
title: "Solution to Bornhack 2020 CTF challenge alice_bob_playing_telepathy"
author: capitol
category: ctf
---

![moon](/images/moon.jpeg)

##### Name:
alice_bob_playing_telepathy

##### Category:
crypto

##### Points:
400

#### Writeup

This was a challenge centered around attacking the lua runtime in a C program.

We got this program

```C
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#include <lauxlib.h>
#include <lua.h>
#include <lualib.h>
#include <string.h>

#include <assert.h>

ssize_t readn(int fd, char *buf, size_t n){
    size_t off = 0;
    while(off < n){
        ssize_t res = read(fd, &buf[off], n-off);
        if(res <= 0) return res;
        off += res;
    }
    return off;
}

int main(int argc, char **argv){
    alarm(30);

    setbuf(stdin, NULL);
    setbuf(stdout, NULL);
    setbuf(stderr, NULL);
    puts("Give me some Alice and Bob:");

    char program[256 + 1] = {0};
    assert(readn(STDIN_FILENO, program, sizeof(program)-1) == sizeof(program)-1);

    lua_State *alice = luaL_newstate();
    //luaL_openlibs(alice); is too powerful
    luaL_requiref(alice, "math", luaopen_math, 1);
    luaL_requiref(alice, "table", luaopen_table, 1);
    luaL_dostring(alice, program);

    lua_State *bob = luaL_newstate();
    //luaL_openlibs(bob); is too powerful
    luaL_requiref(bob, "math", luaopen_math, 1);
    luaL_requiref(bob, "table", luaopen_table, 1);
    luaL_dostring(bob, program);

    int urandom = open("/dev/urandom", O_RDONLY);
    assert(urandom > 0);

    for(int i = 0; i < 1000; i++){
        unsigned int num = 0;
        assert(readn(urandom, (char *)&num, sizeof(num)) == sizeof(num));
        num = num % 100;

        // alice(num)
        lua_getglobal(alice, "alice");
        lua_pushnumber(alice, (float)num);
        assert(lua_pcall(alice, 1, 0, 0) == 0);

        // assert(num == bob())
        lua_getglobal(bob, "bob");
        assert(lua_pcall(bob, 0, 1, 0) == 0);
        assert(lua_isnumber(bob, -1));
        assert(num == (int)lua_tonumber(bob, -1));
        lua_pop(bob, 1);
    }

    lua_close(alice);
    lua_close(bob);

    puts(
#include "flag.h"
    );

}
```

It initializes 2 different `lua_State` objects from the same source code that the attacker supplies.
Source that should have one function `alice` that accepts a parameter, and another function `bob`
that should return a number.

It then generates a series of 1000 random numbers between 0 and 100 and sends to the `alice` function
in the first `lua_State` and reads a number from the `bob` function in the second `lua_State`.

This means that we need to communicate between functions in different `lua_State`.

That the lua math module is loaded gave us a hint, and the random number generator seed turns out to
not be part of the `lua_State` object in lua 5.3 and below, this security hole has been fixed in 5.4.

We wrote this function to reverse the relationship between the randomseed and the random function.

```lua

function alice(n)
  math.randomseed(0)
  for i=0,n do
    math.random()
  end
end
function bob()
  a = math.random()
  math.randomseed(0)
  for i=0,101 do
    if math.random() == a then
        return i-1
    end
  end
end
```

Sadly the CTF closed just before we sent this in, so we didn't manage to get the flag, and could only
verify this locally.
