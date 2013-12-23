SDL iOS build script
===
The script creates a set of frameworks that can be easily used to develop SDL applications for the iOS platform.

Depends on
---
__tools__, you could install them with [homebrew][] `brew install hg`

- hg
- git

__rubygems__, you could install them with bundler `gem install bundler && bundle install`

- rake
- colorize

What frameworks are available?
---
- SDL
- SDL_image
- SDL_mixer
- Tremor *(SDL_mixer contains its own copy now)*

How to use?
---

`rake` or `rake SDL:build` to download sources and build SDL.framework

`rake build_all` to download and build all sdl specific frameworks

More information
---
`rake -T`

License
---
SDL iOS build script is provided under the terms of the [Zlib][license] license

[homebrew]:http://mxcl.github.com/homebrew
[license]:http://www.opensource.org/licenses/Zlib

