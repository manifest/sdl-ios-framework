SDL iOS build script
===
A script creates a pseudo-framework which could be used to build iOS applications for both simulator and device

Depends on
---
__tools__

- hg
- git
- svn

__rubygems__

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
SDL iOS build script is provided under the terms of the [the MIT license][licence]

[licence]:http://www.opensource.org/licenses/mit-license.php

