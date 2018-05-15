### FFmpeg Build with FFplay
* sudo apt install libavcodec-dev
* sudo apt install libmp3lame-dev
* sudo apt-get install libsdl2-dev
* Download ` nasm ` or ` yasm ` and build from source
* [yasm](http://yasm.tortall.net/Download.html)
* [nasm](http://www.nasm.us/)
* Add Path Variable for ` yasm `
* Build FFmpeg
```sh
./configure --enable-ffplay --enable-libmp3lame
make
```
