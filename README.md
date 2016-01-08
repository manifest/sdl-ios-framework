# SDL iOS build script

The script creates iOS pseudo-frameworks for SDL2 and related libraries.

### Depends on

__tools__, you could install them with [homebrew][homebrew] `brew install hg`

- hg
- git

__rubygems__, you could install them with bundler `gem install bundler && bundle install`

- rake
- colorize

### What frameworks are available?

- SDL2
- SDL2_image
- SDL2_mixer
- SDL2_ttf
- Tremor *(SDL2_mixer contains its own copy now)*

### How to use?

`rake` or `rake SDL2:build` to download sources and build SDL2.framework.  
`rake build_all` to download and build all sdl specific frameworks.

### More information

`rake -T`

### License

The SDL iOS build script is provided under the terms of the [Zlib][license] license.

[homebrew]:http://mxcl.github.com/homebrew
[license]:http://www.opensource.org/licenses/Zlib

