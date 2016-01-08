# ------------------------------------------------------------------------------
# Builds a SDL framework for the iOS
# Copyright (c) 2011-2016 Andrei Nesterov
# See LICENSE for licensing information
# ------------------------------------------------------------------------------
#
# The script creates iOS pseudo-frameworks for SDL2 and related libraries
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
#   Archs       a list of architectures planned to support (e.g. [:armv6,:i386])
#   Conf        configuration (:release or :debug)
#   SDK         a version number of installed iOS SDK (e.g. "4.3")
#
# ------------------------------------------------------------------------------

require 'rubygems'
require 'colorize'
require 'find'

# --- Configure ----------------------------------------------------------------
module Configure
	Archs = [:armv7, :armv7s, :arm64, :i386, :x86_64]
	Conf = :release
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
		psdk, parch = compute_platform(sdk, arch)

		args = []
		args << %{-sdk "#{psdk}"}
		args << %{-configuration "#{conf}"}
		args << %{-target "#{target}"}
		args << %{-arch "#{parch}"}
		args << %{-project "#{project}"}
		args << %{TARGETED_DEVICE_FAMILY="1,2"}
		args << %{BUILT_PRODUCTS_DIR="build"}
		args << %{CONFIGURATION_BUILD_DIR="#{dest}"}
		args << %{CONFIGURATION_TEMP_DIR="#{dest}.build"}

		case arch
			when :arm64
				args << %{IPHONEOS_DEPLOYMENT_TARGET=7.0}
			when :x86_64
				args << %{VALID_ARCHS="x86_64"}
				args << %{IPHONEOS_DEPLOYMENT_TARGET=6.0}
			else
				args << %{IPHONEOS_DEPLOYMENT_TARGET=5.0}
		end

		refresh_dir dest
		Dir.chdir File.dirname(project) do
			system %{xcodebuild #{args.join " "}}
		end
	end

	def build_framework_library(outputPath, inputPaths)
		refresh_dir(File.dirname(outputPath))
		system %{lipo -create #{inputPaths.join(" ")} -o #{outputPath}}
	end

	def build_framework(name, version, identifier, dest, headers, project, target, conf, sdk, archs)
		ulibDir = "#{dest}/universal-#{conf}"
		libPaths = []
		archs.each do |arch|
			build_library(project, dest, target, conf, sdk, arch)

			Find.find(library_bundle_path(dest, conf, sdk, arch)) do |path| 
				libPaths.push(path) if path =~ /\A.*\.a\z/
			end
		end

		ulibPath = "#{ulibDir}/#{File.basename(libPaths.first)}"
		build_framework_library(ulibPath, libPaths)
		create_framework(name, version, identifier, dest, headers, ulibPath)
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
		psdk, parch = compute_platform(sdk, arch)
		"#{path}/#{psdk}-#{parch}-#{conf}"
	end

	def framework_bundle_path(path, name)
		"#{path}/#{name}.framework"
	end

	def compute_platform(sdk, arch)
		case arch
			when String;         [sdk, arch]
			when :i386, :x86_64; ["iphonesimulator" + sdk, arch.to_s]
			else                 ["iphoneos" + sdk, arch.to_s]
		end
	end

	private :library_bundle_path, :framework_bundle_path, :compute_platform 
end

# --- Package ------------------------------------------------------------------
class Package
	extend Builder

	def self.build(conf, sdk, archs)
		download
		unpack
		build_phase(config(conf), sdk, archs)
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
	Version = "2.0.4"

	def self.download
		message "downloading SDL"
		system %{hg clone -u release-#{Version} "http://hg.libsdl.org/SDL" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, archs)
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
			archs
		)
	end
end

# --- SDL_image ----------------------------------------------------------------
class SDL2_image < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_image"
	BuildDir = "#{Global::BuildDir}/sdl_image"
	Version = "2.0.1"
 
	def self.download
		message "downloading SDL_image"
		system %{hg clone -u release-#{Version} "http://hg.libsdl.org/SDL_image" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, archs)
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
			archs
		)
	end
end

# --- SDL_ttf ----------------------------------------------------------------
class SDL2_ttf < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_ttf"
	BuildDir = "#{Global::BuildDir}/sdl_ttf"
	Version = "2.0.13"

	def self.download
		message "downloading SDL_ttf"
		system %{hg clone -u release-#{Version} "http://hg.libsdl.org/SDL_ttf" "#{SourcesDir}"}
	end

	def self.build_phase(conf, sdk, archs)
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
			archs
		)
	end
end

# --- SDL_mixer ----------------------------------------------------------------
class SDL2_mixer < Package
	SourcesDir = "#{Global::SourcesDir}/sdl_mixer"
	BuildDir = "#{Global::BuildDir}/sdl_mixer"
	Version = "2.0.1"

	def self.download
		message "downloading SDL_mixer"
		system %{hg clone -u release-#{Version} "http://hg.libsdl.org/SDL_mixer" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, archs)
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
			archs
		)
	end
end

# --- Tremor -------------------------------------------------------------------
class Tremor < Package
	SourcesDir = "#{Global::SourcesDir}/cocos2d"
	BuildDir = "#{Global::BuildDir}/tremor"
	Version = "1.3.2"

	def self.download
		message "downloading Cocos2d"
		system %{git clone "https://github.com/cocos2d/cocos2d-iphone.git" "#{SourcesDir}"}
	end
	
	def self.build_phase(conf, sdk, archs)
		message "switching to v1.x brunch"
		Dir.chdir(SourcesDir) do
			system 'git checkout release-1.1'
		end

		message "building Tremor"
		self.build_framework(
			"vorbis",
			Version,
			"org.xiph",
			BuildDir, 
			"#{SourcesDir}/external/Tremor",
			"#{SourcesDir}/cocos2d-ios.xcodeproj",
			"vorbis",
			conf,
			sdk,
			archs
		)
	end
end

# --- Tasks --------------------------------------------------------------------
require 'rake/clean'

SDLL = [:SDL2, :SDL2_image, :SDL2_mixer, :SDL2_ttf]
AllL = SDLL + [:Tremor]

desc "Download and build the SDL framework"
task :default => ["SDL2:build"] do
end

desc "Download and build all SDL specific frameworks: [#{SDLL.join ", "}]"
task :build_all do
	SDLL.each do |classname|
		Object.const_get(classname).build Configure::Conf, Configure::SDK, Configure::Archs
	end
end

AllL.each do |classname|
	namespace classname do
		obj = Object.const_get(classname)

		desc "#{classname}: download and build framework"
		task :build do
			obj.build Configure::Conf, Configure::SDK, Configure::Archs
		end

		desc "#{classname}: download"
		task :download do
			obj.download
			obj.unpack
		end

		desc "#{classname}: build framework"
		task :build_phase do
			obj.build_phase Configure::Conf, Configure::SDK, Configure::Archs
		end

		desc "#{classname}: remove any generated file"
		task :clobber do
			CLOBBER = Rake::FileList.new
			CLOBBER.include obj::SourcesDir, obj::BuildDir
			Rake::Task[:clobber].execute
		end
	end
end

CLOBBER.include Global::SourcesDir, Global::BuildDir

