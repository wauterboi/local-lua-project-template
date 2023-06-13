# Local LuaRocks-based template

Lots of people want LuaRocks to function like `npm`. While LuaRocks provides
some playful ideas for supporting this, I'm taking a stab with wrapper scripts.

# Requirements

* `sh`
* `luarocks`

# Explanation

Lua looks for modules in according to `LUA_PATH` and `LUA_CPATH`. All I'm doing
is setting that with a couple shell scripts. I'm sure adapting this for Windows
is simple, but as I never use PowerShell or batch files for anything, this
project is restricted to Linux. In my case, I'm using Windows Subshell for
Linux.

# `LUA_PATH` and `LUA_CPATH` values

This is where the supplied wrappers will instruct Lua to search for modules.

* `LUA_PATH`
	* `<project_path>/lua/?.lua`
	* `<project_path>/lua/?/init.lua`
	* `<project_path>/lua_modules/share/lua/5.1/?.lua`
	* `<project_path>/lua_modules/share/lua/5.1/?/init.lua`
	* `$LUA_PATH`
* `LUA_CPATH`
	* `<project_path>/lua_modules/lib/lua/5.1/?.so;`
	* `$LUA_CPATH`

For more information on these environment variables, check out the Lua manual - specifically the sections explaining [`require`](https://www.lua.org/manual/5.1/manual.html#pdf-require), [`package.path`](https://www.lua.org/manual/5.1/manual.html#pdf-package.path), and [`package.cpath`](https://www.lua.org/manual/5.1/manual.html#pdf-package.cpath).

# The `bin` folder

The meat and potatoes of this project are the wrapper scripts `bin/env` and
`bin/luarocks`, which automatically set `LUA_PATH` and `LUA_CPATH` to use
the project-supplied modules before system modules.

## `bin/env`
This script automatically prepends project directories to the environment
variables `LUA_PATH` and `LUA_CPATH` before executing all arguments after as a
separate command. For example: `bin/env lua51 -e "print(package.path)"` will
print the project's `LUA_PATH` value.

You could also run `busted` and other binaries that rely on `LUA_PATH` and
`LUA_CPATH` within the environment using `bin/env busted`. Obviously, if you
have installed binaries through Luarocks in the project, you will need to
reference its location in the `lua_modules` folder.

## `bin/luarocks`
This script calls `bin/env` automatically and calls
`/usr/bin/env luarocks --tree "<project_path>/lua_modules>"`.
For example, if you wanted to download all dependencies, use
`bin/luarocks build --only-deps`.

# The Rockspec

This project comes with a simple `nil-dev-1.rockspec` file. Obviously, the
filename and contents should be updated to reflect the project and its
dependencies. Unfortunately, LuaRocks does not support functionality similar
to `npm install <package_name>` with the `--save` or `--save-dev` flags,
so the dependencies need to be
[added by hand](https://github.com/luarocks/luarocks/wiki/Rockspec-format).