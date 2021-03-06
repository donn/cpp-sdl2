# cpp-sdl2 
[![linux](https://github.com/Edhebi/cpp-sdl2/workflows/linux/badge.svg)][linux-badge]
[![windows](https://github.com/Edhebi/cpp-sdl2/workflows/windows/badge.svg)][windows-badge]
[![doc](https://github.com/Edhebi/cpp-sdl2/workflows/doc/badge.svg)][doc-badge]

[linux-badge]: https://github.com/Edhebi/cpp-sdl2/actions?query=workflow%3Alinux
[windows-badge]: https://github.com/Edhebi/cpp-sdl2/actions?query=workflow%3Awindows
[doc-badge]: https://edhebi.github.io/cpp-sdl2/doc/

Basic C++17 bindings to [SDL2], implemented as an header-only library

[SDL2]: https://wiki.libsdl.org/FrontPage

## Documentation

This project uses doxygen for its documentation, you can find it [here](https://edhebi.github.io/cpp-sdl2/doc).

## Usage

This library has been written in C++17. It should work out of the box with any modern compiler, but you may need to set
some flags ([gcc][gcc-c++17], [clang][clang-c++17], [msvc][msvc-c++17]). If you use the cmake target, this will be done
automatically.

[gcc-c++17]: https://gcc.gnu.org/projects/cxx-status.html#cxx17
[clang-c++17]: https://clang.llvm.org/cxx_status.html#cxx17
[msvc-c++17]: https://docs.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version

This library is header-only, meaning that there is no build step. You only need to make the `sources` directory visible
to your compiler, and replace your `SDL.h` include by `#include <cpp-sdl2/sdl.hpp`. You still need to have SDL2 be
visible and properly linked.

## Configuration flags

Some SDL2 features require external libraries. Since we want to be as compatible as possible out of the box, those are
behind `#define` flags. Those flags must be defined before any include to the library.
 
You can either define those flags yourself, or use the cmake options with the same name, which will add those flags to
your compiler definitions list.

### CPP_SDL2_ENABLE_OPENGL

This flag enables opengl features around windows. Note that you still need an opengl loader, like [glew], [gl3w],
[glad], or [epoxy].

[glew]: http://glew.sourceforge.net/
[gl3w]: https://github.com/skaslev/gl3w
[glad]: https://github.com/Dav1dde/glad
[epoxy]: https://github.com/anholt/libepoxy

### CPP_SDL2_ENABLE_VULKAN

This flag enable vulkan features around window. It's intended to be uses with nvidia's (now standard) `vulkan.hpp`
header. You still need a vulkan runtime and a recent version of the vulkan sdk.

### CPP_SDL2_ENABLE_SDL_IMAGE

This uses the SDL2_Image library to load surfaces more easily. You still need to have the library installed and visible.

### CPP_SDL2_ENABLE_SDL_TTF

This uses the SDL2_TTF library to load fonts more easily. You still need to have the library installed and visible.

### CPP_SDL2_DISABLE_EXCEPTIONS

`cpp-sdl2` uses exceptions very conservatively, and most of them indicate a failure that it probably not recoverable.
Correctly used, this library shouldn't throw *any* exception. If you still need to disable exceptions, this flag will
replace exceptions by a log to `stderr` followed by an `abort()`.

## Dependencies

### Mandatory

- [SDL2]
- A C++17 capable compiler

### Optional

- SDL2_image
- SDL_ttf
- OpenGL
- Vulkan

SDL versions from `2.0.8` to `2.0.12` have been tested and should work.

## Example

```cpp
#include <cpp-sdl2/sdl.hpp>
#include <iostream>

#include <cstdlib> // Using C-style rand
#include <ctime>

int main([[maybe_unused]] int argc, [[maybe_unused]] char* argv[])
{
    std::srand(static_cast<unsigned>(std::time(nullptr)));
    
    // The following objects manages the lifetime of SDL resources RAII style
    
    sdl::Root root{SDL_INIT_EVENTS};
    sdl::Window window{"Random Colors", {600, 600}};
    
    sdl::Renderer renderer = window.make_renderer();
    
    sdl::Color background = sdl::Color::Black();
    
    bool done = false;
    bool redraw = true;
    
    while (!done)
    {
        if (redraw)
        {
            renderer.clear(background);
            renderer.present();
            redraw = false;
        }
        
        sdl::Event event;
        while (event.pull())
        {
            if (event.type == SDL_QUIT) done = true;
            
            if (event.type == SDL_MOUSEBUTTONUP)
            {
                background.r = std::rand() % 256;
                background.g = std::rand() % 256;
                background.b = std::rand() % 256;
                redraw = true;
            }
        }
    }

    // Cleanup is done automatically
    return 0;
}
```
