SDL iOS build script
===
A script creates a pseudo-framework which could be used to build iOS applications for both simulator and device

Depends on
---
__tools__, you could install them with [homebrew][] `brew install hg`

- hg
- git
- svn

__rubygems__, you could install them with `sudo gem install colorize`

- colorize

What frameworks are available?
---
- SDL
- SDL_image
- SDL_mixer
- Tremor

How to use?
---
`rake` or `rake SDL:build` to download sources and build SDL.framework

`rake build_all` to download and install all available frameworks

More documentation
---
`rake -T`

License
---
SDL iOS build script is provided under the terms of the [the Zlib license][license]

[homebrew]:http://mxcl.github.com/homebrew
[license]:http://www.opensource.org/licenses/Zlib

