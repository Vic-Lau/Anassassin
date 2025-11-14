# Anassassin


=============================================================
flameshot:

env:
Qt6_DIR	D:\Qt\6.7.3\msvc2022_64\lib\cmake\Qt6
PROJ	D:\work\flameshot
BUILD_DIR	D:\work\flameshot\build
INSTALL_PREFIX	D:\work\flameshot\_install

build:
cd D:\work\flameshot
if (Test-Path .\build) { Remove-Item .\build -Recurse -Force }

cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DQt6_DIR="D:\Qt\6.7.3\msvc2022_64\lib\cmake\Qt6" `
  -DCMAKE_PREFIX_PATH="D:\Qt\6.7.3\msvc2022_64" `
  -DQT_DEBUG_FIND_PACKAGE=ON

cmake --build build --config Release

copy dll:
& "D:\Qt\6.7.3\msvc2022_64\bin\windeployqt.exe" `
  --release `
  --dir "D:\work\flameshot\build\src\Release" `
  "D:\work\flameshot\build\src\Release\flameshot.exe"

run:
D:\work\flameshot\build\src\Release\flameshot.exe
D:\work\flameshot\build\src\Release\flameshot.exe gui
D:\work\flameshot\build\src\Release\flameshot-cli.exe -h
D:\work\flameshot\build\src\Release\flameshot-cli.exe gui -d 1000

=============================================================
