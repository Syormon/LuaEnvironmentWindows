# LuaEnvironmentWindows
Building an environment for lua in windows around a VS Code extension

We will use Luarocks as a PM with MSYS2 gcc (mingw64)

## 1. MSYS2 installation

1. Visit https://www.msys2.org/
2. Click on the link next to "Download the installer"
3. Run the installer
4. Add `C:\msys64\usr\bin\` to PATH
5. Add `C:\msys64\mingw64\bin` to PATH
6. Open mingw64
7. Run `pacman -S mingw64/mingw-w64-x86_64-gcc`
8. Confirm with enter key
9. Run `pacman -Syu` (This might close out of mingw64 if there is a kernel level update)
10. Run the following: <br> `pacman --noconfirm -S mingw-w64-x86_64-gcc mingw-w64-x86_64-cmake` <br> `pacman --noconfirm -S mingw-w64-x86_64-extra-cmake-modules make tar` <br> `pacman --noconfirm -S mingw64/mingw-w64-x86_64-cyrus-sasl` <br>

## 2. Lua installation

1. Visit http://www.lua.org/
2. Click on `download`
3. Click on `get a binary`
4. Click on `download`
5. Click on `lua-\<version\>_Win64_bin.zip` e.g. `lua-5.3.6_Win64_bin.zip` or `lua5_3_6_Win64_bin.zip`
6. Create folder `C:\lua`
7. Extract content into `C:\lua` (exe files should be in C:\lua)
8. Add `C:\lua` to PATH
9. Create `C:\lua\include` and `C:\lua\lib` folders
10. Return to the lua download page in your browser
11. Click on `LuaBinaries \<version\> - Release X`
12. Navigate to `Windows Libraries\Dynamic`
13. Download `lua-\<version\>_Win64_dllw6_lib.zip`
14. Copy include files into `C:\lua\include`
15. Copy lib files into `C:\lua\lib`

## 3. Luarocks installation

1. Visit https://luarocks.org/
1. Click on "install"
1. Click on "Windows all-in-one executable (64-bit)"
1. Download file
1. Extract exe files into `C:\lua`
1. Add `%appdata%\luarocks\bin` to PATH

## 4. Luarocks config

1. Delete old config: remove luarocks folder in %localappdata% and %appdata%
1. Open cmd.exe
2. Run `luarocks config lua_version 5.3`
3. Run `luarocks config variables.LUA_INCDIR C:\lua\include`
4. Run `luarocks config variables.LUA_LIBDIR C:\lua\lib`
5. Run `luarocks config variables.CC gcc`
6. Run `luarocks config variables.LD gcc`
7. Run `luarocks` -> there should be no config errors left

## 5. Testing

1. Try to install something with luarocks e.g. in cmd.exe run `luarocks install luasocket` -> should compile x64 and install without errors

## 6. Visual Studio Code

1. Open up VS code
2. install [Lua Debug by actboy168](https://marketplace.visualstudio.com/items?itemName=actboy168.lua-debug) (this includes 64bit lua debugger in 5.3)
3. hit the gear and open up extension settings
4. change 'Lua Version' to 5.3
5. open up settings.json
6. add cpath as following: <br> `"lua.debug.settings.cpath": "C:/Users/useradmin/AppData/Roaming/luarocks/lib/luarocks/lib/lua/5.3/?.dll"`
7. add path as following: <br> `"lua.debug.settings.path": ""C:/Users/useradmin/AppData/Roaming/luarocks/lib/luarocks/share/lua/5.3/?.lua"`

## 7. Visual Studio

1. Install Visual studio <https://visualstudio.microsoft.com/vs/community/>
2. make sure c/c++ binaries are added in the visual studio installer
3. You should see `Developer Command Prompt for VS XXXX` installed. This will be useful for cross-compiling against the windows-sdk. 

## 8. Compiling specific packages

| Package | Build instructions |
| ------- | ------------------ |
| lua-cURLv3    | 1. make sure you have the curl binaries (Download curl 64-bit https://curl.se/windows/), <br> 2. add include directory in fetch like `luarocks install Lua-cURL CURL_DIR=C:\lua\curl` <br> 3.Move `libcurl-x64.dll` from the `C:\lua\curl\bin` directory to `C:\lua` directory.|
| dkjson  | `luarocks install dkjson` |
| lpeg    | `luarocks install lpeg` |
| lua-zip | 1. download libzip from https://libzip.org/download/<br> 2.	place it in c:\msys64\home\<USER>\ <br> 3. extract the tar by deflate using `tar xzf <filename>`<br> 4. open up mingw64 and navigate to the folder you just extracted to (in my case libzip-1.9.2)<br> 5. create the following directory and navigate to it `mkdir cmake-build && cd cmake-build`<br> 6.	run the following cmake command: cmake -G "MSYS Makefiles" ..<br> 7. open up Developer Command Prompt for VS in administrator mode<br> 8.	run the cmake command: `cmake --build . --target install`<br> 9. this will build libzip for windows and install in your /Program Files (x86)/ directory. open up cmd/powershell and run the following: `luarocks install lua-zip ZIP_DIR="C:\Program Files (x86)\libzip"` <br> 10. Move file from `\libzip\bin\libzip.dll` to `c:\lua` folder to allow environment PATH variable to find it. |
| LuaExpat | 1. download expat SAX binaries from https://github.com/libexpat/libexpat/releases (in tar.gz format) <br> 2.	place it in c:\msys64\home\<USER>\ <br> 3. extract the tar by deflate using `tar xzf <filename>`<br> 4. open up mingw64 and navigate to the folder you just extracted to (in my case expat-2.5.0)<br> 5. create the following directory and navigate to it `mkdir cmake-build && cd cmake-build`<br> 6.	run the following cmake command: cmake -G "MSYS Makefiles" ..<br> 7. open up Developer Command Prompt for VS in administrator mode<br> 8.	run the cmake command: `cmake --build . --target install`<br> 9. this will build expat for windows and install in your /Program Files (x86)/ directory. open up cmd/powershell and run the following: `luarocks install lua-zip EXPAT_DIR="C:\Program Files (x86)\expat"` |
| luafilesystem | `luarocks install luafilesystem` |
| luasocket | `luarocks install luasocket` |
| Luasec | 1.	Install openssl using the mingw64 terminal: `pacman -S mingw-w64-x86_64-openssl` and `pacman -S mingw-w64-x86_64-libgcrypt`<br> 2.	Manually download the .rockspec file and open it up in vs code <br> 3.	In line 90, there is a table called properties. Remove M32D from the suffixes so it looks like this: `libraries = {"libssl", "libcrypto", "ws2_32" }`<br> 4.	`luarocks install luasec-1.2.0-1.rockspec OPENSSL_DIR=C:\msys64\mingw64` |
| lua-mimetypes | `luarocks install mimetypes` |
| lua-cjson | `luarocks install lua-cjson "CFLAGS=-DLUA_COMPAT_5_1"` |
| rapidjson | `luarocks install rapidjson LUA_LIBDIR=C:/lua/lib/` |
| lua-mongo | 1. open up mingw64 <br> 2. `wget https://github.com/mongodb/mongo-c-driver/releases/download/1.17.5/mongo-c-driver-1.17.5.tar.gz` <br> 3. `tar xzf mongo-c-driver-1.17.5.tar.gz` <br> 4. `cd mongo-c-driver-1.17.5` <br> 5. `mkdir cmake-build && cd cmake-build` <br> 6. `cmake -G "MSYS Makefiles" -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF ..` <br> 7. Once it finishes generating the makefile, open up developer  command prompt for VS in admin mode. <br> 8. Navigate to the cmake-build directory, and then install: `cmake --build . --target install` <br> 9. This will install to the `C:\Program Files (x86)\mongo-c-driver` directory. You can make sure the install worked by running one of the example-xxx.exe's in the `\cmake-build\src\` folder. <br> 10. `luarocks install lua-mongo LIBBSON_DIR="C:\Program Files (x86)\mongo-c-driver" LIBMONGOC_DIR="C:\Program Files (x86)\mongo-c-driver"` <br> 11. Move `libmongoc-1.0.dll` and `libbson-1.0.dll` from your `C:\Program Files (x86)\mongo-c-driver\bin` directory to your `C:\lua` directory. |
| luz-zlib | https://zlib.net/zlib1213.zip - Working with Developer to fix their rockspec currently. |
| luasql | 1. open up mingw64 <br> 2. `pacman -S mingw-w64-x86_64-unixodbc` <br> 3. Open up cmd/powershell in administrator <br> 4. `luarocks install luasql-odbc ODBC_DIR=C:\msys64\mingw64` 
