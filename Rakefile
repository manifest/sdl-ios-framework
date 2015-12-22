# ------------------------------------------------------------------------------
# Builds a SDL framework for the iOS
# Copyright (c) 2011-2014 Andrei Nesterov
# See LICENSE for licensing information
# ------------------------------------------------------------------------------
#
# The script creates a set of pseudo-frameworks that can be easily used to develop
# a SDL applications for the iOS platform.
#
# ------------------------------------------------------------------------------
#
# Depends on tools:
#  hg
#  git
#
# Depends on rubygems:
#  rake
#  colorize
#
# ------------------------------------------------------------------------------
#
# To configure the script, define:
#   Conf      	   configuration (:release or :debug)
#   Arch      	   architecture for the device's library (e.g. :armv6)
#   SDK       	   version number of the iOS SDK (e.g. "4.3")
#
# ------------------------------------------------------------------------------

require 'rubygems'
require 'colorize'
require 'find'

# --- Configure ----------------------------------------------------------------
module Configure
	Conf = :release
	Arch = [:armv7, :armv7s, :arm64, :i386, :x86_64]
	SDK = `xcrun --sdk iphoneos --show-sdk-version`.chop
end

# --- Constants ----------------------------------------------------------------
module Global
	RootDir = Dir.pwd
	TemplatesDir = "#{RootDir}/templates"
	SourcesDir = "#{RootDir}/src"
	BuildDir = "#{RootDir}/build"
end

# --- Common -------------------------------------------------------------------
def message(text)
	tailSize = 75 - text.length;
	puts "\n>>> #{text} #{'-' * (tailSize < 0 ? 0 : tailSize)}".green
end

def refresh_dir(path)
	system %{ rm -rf #{path} } if FileTest.exists? path
	system %{ mkdir -p #{path} }
end

module Builder
	def build_library(project, dest, target, conf, sdk, arch)
		dest = library_bundle_path(dest, conf, sdk, arch)
		sdk, arch = compute_platform(sdk, arch)

		args = []
		args << %{-sdk "#{sdk}"}
		args << %{-configuration "#{conf}"}
		args << %{-target "#{target}"}
		args << %{-arch "#{arch.to_str}"}
		args << %{-project "#{project}"}
		args << %{TARGETED_DEVICE_FAMILY="1,2"}
		args << %{BUILT_PRODUCTS_DIR="build"}
		args << %{CONFIGURATION_BUILD_DIR="#{dest}"}
		args << %{CONFIGURATION_TEMP_DIR="#{dest}.build"}

# this logic is suggested here: http://blog.diogot.com/blog/2013/09/18/static-libs-with-support-to-ios-5-and-arm64/
        target = '5.0'
        if arch == 'arm64'
            target = '7.0'
        elsif arch == 'x86_64'
            target = '6.0'
            args << %{VALID_ARCHS="x86_64"}
        end
		args << %{IPHONEOS_DEPLOYMENT_TARGET="#{target}"}

		refresh_dir dest
		Dir.chdir File.dirname(project) do
			system %{xcodebuild #{args.join " "}}
		end
	end

	def build_framework_library(frameworkLib, name, dest, conf, sdk, arch)
        libs = ''
        for a in arch
             deviceLib = "#{library_bundle_path(dest, conf, sdk, a)}/#{name}"
             libs << deviceLib << ' ';
        end
    	refresh_dir(File.dirname(frameworkLib))
		system %{lipo -create #{libs} -o #{frameworkLib}}
	end

	def build_framework(name, version, identifier, dest, headers, project, target, conf, sdk, arch)
        for a in arch
    		build_library(project, dest, target, conf, sdk, a)
        end

		libFileName = nil
		Find.find(library_bundle_path(dest, conf, sdk, arch.at(0))) do |path| 
			libFileName = File.basename(path) if path =~ /\A.*\.a\z/
		end
		libFilePath = "#{dest}/universal-#{conf}/#{libFileName}"

		build_framework_library(libFilePath, libFileName, dest, conf, sdk, arch)
		create_framework(name, version, identifier, dest, headers, libFilePath)
	end

	def create_framework(name, version, identifier, dest, headers, lib)
		frameworkVersion = "A"
		frameworkBundle = framework_bundle_path(dest, name)

		# creating framework's directories	
		refresh_dir frameworkBundle
		system %{ mkdir "#{frameworkBundle}/Versions" }
		system %{ mkdir "#{frameworkBundle}/Versions/#{frameworkVersion}" }
		system %{ mkdir "#{frameworkBundle}/Versions/#{frameworkVersion}/Resources" }
		system %{ mkdir "#{frameworkBundle}/Versions/#{frameworkVersion}/Headers" }
		system %{ mkdir "#{frameworkBundle}/Versions/#{frameworkVersion}/Documentation" }

		# creating framework's symlinks
		system %{ ln -s "#{frameworkVersion}" "#{frameworkBundle}/Versions/Current" }
		system %{ ln -s "Versions/Current/Headers" "#{frameworkBundle}/Headers" }
		system %{ ln -s "Versions/Current/Resources" "#{frameworkBundle}/Resources" }
		system %{ ln -s "Versions/Current/Documentation" "#{frameworkBundle}/Documentation" }
		system %{ ln -s "Versions/Current/#{File.basename lib}" "#{frameworkBundle}/#{name}" }

		# copying lib
		system %{ cp "#{lib}" "#{frameworkBundle}/Versions/#{frameworkVersion}" }

		# copying includes
		FileList["#{headers}/*.h"].each do |source|
			system %{ cp "#{source}" "#{frameworkBundle}/Headers" }
		end

		# creating plist
		File.open("#{frameworkBundle}/Resources/Info.plist", "w") do |f|
			f.puts '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">'
			f.puts "<dict>
	<key>CFBundleDevelopmentRegion</key>
	<string>Englisystem</string>
	<key>CFBundleExecutable</key>
	<string>#{name}</string>
	<key>CFBundleIdentifier</key>
	<string>#{identifier}</string>
	<key>CFBundleInfoDictionaryVersion</key>
	<string>6.0</string>
	<key>CFBundlePackageType</key>
	<string>FMWK</string>
	<key>CFBundleSignature</key>
	<string>????</string>
	<key>CFBundleVersion</key>
	<string>#{version}</string>
</dict>
</plist>"
		end
	end

	def library_bundle_path(path, conf, sdk, arch)
		sdk, arch = compute_platform(sdk, arch)
		"#{path}/#{sdk}-#{arch}-#{conf}"
	end

	def framework_bundle_path(path, name)
		"#{path}/#{name}.framework"
	end

	def compute_platform(sdk, arch)
		return [sdk, arch] if arch.class == String
		[arch == :i386 || arch == :x86_64 ? "iphonesimulator" + sdk : "iphoneos" + sdk, arch.to_s]
	end

	private :library_bundle_path, :framework_bundle_path, :compute_platform 
end

# --- Package ------------------------------------------------------------------
class Package
	extend Builder

	def self.build(conf, sdk, arch)
		download
		unpack
		build_phase(config(conf), sdk, arch)
	end

	def self.unpack
	end

	def self.config conf
		conf == :release ? "Release" : "Debug"
	end
end

# --- SDL ----------------------------------------------------------------------
class SDL2 < Package
	ProjFile = "SDL.xcodeproj"
	SourcesDir = "#{Global::SourcesDir}/SDL"
	BuildDir = "#{Global::BuildDir}/sdl"
	Version = "2.0.4hg"

	def self.download
		message "downloading SDL"
		system %{hg clone "http://hg.libsdl.org/SDL" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, arch)
		message "building SDL"
		self.build_framework(
			"SDL2",
			Version,
			"org.libsdl",
			BuildDir, 
			"#{SourcesDir}/include",
			"#{SourcesDir}/Xcode-iOS/SDL/#{ProjFile}",
			"libSDL",
			conf,
			sdk,
			arch
		)
	end
end

# --- SDL_image ----------------------------------------------------------------
class SDL2_image < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_image"
	BuildDir = "#{Global::BuildDir}/sdl_image"
	Version = "2.0.0hg"
 
	def self.download
		message "downloading SDL_image"
		system %{hg clone "http://hg.libsdl.org/SDL_image" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, arch)
		message "building SDL_image"
		self.build_framework(
			"SDL2_image",
			Version,
			"org.libsdl",
			BuildDir, 
			"#{SourcesDir}",
			"#{SourcesDir}/Xcode-iOS/SDL_image.xcodeproj",
			"libSDL_image",
			conf,
			sdk,
			arch
		)
	end
end

# --- SDL_ttf ----------------------------------------------------------------
class SDL2_ttf < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_ttf"
	BuildDir = "#{Global::BuildDir}/sdl_ttf"
	Version = "2.0.12hg"

	def self.download
		message "downloading SDL_ttf"
		system %{hg clone "http://hg.libsdl.org/SDL_ttf" "#{SourcesDir}"}
	end

	def self.build_phase(conf, sdk, arch)
		message "building SDL_ttf"
		self.build_framework(
			"SDL2_ttf",
			Version,
			"org.libsdl",
			BuildDir,
			"#{SourcesDir}",
			"#{SourcesDir}/Xcode-iOS/SDL_ttf.xcodeproj",
			"Static Library",
			conf,
			sdk,
			arch
		)
	end
end

# --- SDL_mixer ----------------------------------------------------------------
class SDL2_mixer < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_mixer"
	BuildDir = "#{Global::BuildDir}/sdl_mixer"
	Version = "2.0.0hg"

	def self.download
		message "downloading SDL_mixer"
		system %{hg clone "http://hg.libsdl.org/SDL_mixer" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, arch)
		message "patching SDL_mixer"
		system %{sed -e 's/#include <endian.h>/#include <machine\\/endian.h>/' -i.back "#{SourcesDir}/external/libvorbisidec-1.2.1/misc.h"}

		message "building SDL_mixer"
		self.build_framework(
			"SDL2_mixer",
			Version,
			"org.libsdl",
			BuildDir, 
			"#{SourcesDir}",
			"#{SourcesDir}/Xcode-iOS/SDL_mixer.xcodeproj",
			"Static Library",
			conf,
			sdk,
			arch
		)
	end
end

# --- Tasks --------------------------------------------------------------------
require 'rake/clean'

SDLL = [:SDL2, :SDL2_image, :SDL2_mixer, :SDL2_ttf]

desc "Download and build the SDL framework"
task :default do
    Object.const_get(:SDL2).build Configure::Conf, Configure::SDK, Configure::Arch
end

desc "Download and build all SDL specific frameworks: [#{SDLL.join ", "}]"
task :build_all do
	SDLL.each do |classname|
		Object.const_get(classname).build Configure::Conf, Configure::SDK, Configure::Arch
	end
end

CLOBBER.include Global::SourcesDir, Global::BuildDir

