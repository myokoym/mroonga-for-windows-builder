require "fileutils"
require "open-uri"
require "archive/zip"
require "octokit"

mariadb_version = "10.0.11"
mroonga_version = "4.03"
repo = "mroonga/mroonga"

def vc_version
  vc_version = "2013"
end

def build_base_name
  "build-vc#{vc_version}"
end

source_name = "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-for-windows"

release_packages = [
  "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-win32.zip",
  "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-win32.msi",
  "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-winx64.zip",
  "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-winx64.msi",
]

task :default => :help

desc "Show help as `rake -T`"
task :help do
  sh("rake", "-T")
end

def download(url, local_path=nil)
  local_path ||= File.basename(url)
  open(url) do |remote_file|
    File.open(local_path, "wb") do |local_file|
      local_file.print(remote_file.read)
    end
  end
  local_path
end

def msi_enabled?
  vc_version == "2010"
end

def vc_formal_version
  case vc_version
  when "2010"
    "Visual Studio 10"
  when "2012"
    "Visual Studio 11"
  when "2013"
    "Visual Studio 12"
  else
    raise "Undefined vc_version: #{vc_version}"
  end
end

def vc_build(type, architecture)
  work_dir = "#{build_base_name}-#{type}-#{architecture}"
  case type
  when "zip"
    target = "package"
  when "msi"
    target = "msi"
  end
  generator_name = vc_formal_version
  generator_name << " Win64" if architecture == "64"
  FileUtils.rm_rf(work_dir)
  FileUtils.mkdir(work_dir)
  FileUtils.chdir(work_dir) do
    sh("cmake ..\\source -G \"#{generator_name}\" > config.log")
    sh("cmake --build . --config RelWithDebInfo > build.log")
    sh("cmake --build . --config RelWithDebInfo --target #{target} > #{type}.log")
    FileUtils.mv("*.#{type}", "..")
  end
end

desc "Download source file from groonga.org"
task :download do
  url = "http://packages.groonga.org/source/mroonga/#{source_name}.zip"
  local_path = File.basename(url)
  puts("Source file: #{url}")
  puts("Downloading...")
  download(url)
end

file "source" do
  puts("Extracting...")
  Archive::Zip.extract("#{source_name}.zip", ".")
  FileUtils.mv(source_name, "source")
end

namespace :build do
  desc "Build all targets"
  task :all => [
    "build:win32:zip",
    "build:winx64:zip",
    "build:win32:msi",
    "build:winx64:msi",
  ]

  namespace :win32 do
    desc "Build win32-zip"
    task :zip => "source" do
      vc_build("zip", "32")
    end

    desc "Build win32-msi"
    task :msi => "source" do
      vc_build("msi", "32")
    end
  end

  namespace :winx64 do
    desc "Build winx64-zip"
    task :zip => "source" do
      vc_build("zip", "64")
    end

    desc "Build winx64-msi"
    task :msi => "source" do
      vc_build("msi", "64")
    end
  end
end

desc "Rename release packages"
task :rename do
  release_packages.each do |package|
    if /.msi\z/ =~ package
      next unless msi_enabled?
    end
    FileUtils.mv(package.sub(/-with-mroonga-#{mroonga_version}/, ""),
                 package)
  end
end

desc "Upload packages to GitHub"
task :upload do
  if ENV["GITHUB_TOKEN"].nil?
    raise "must set GITHUB_TOKEN environment variable"
  end
  access_token = ENV["GITHUB_TOKEN"]
  client = Octokit::Client.new(:access_token => access_token)
  releases = client.releases(repo)
  current_releases = releases.select do |release|
    release.tag_name == "v#{mroonga_version}"
  end
  url = current_releases.first.url
  release_packages.each do |package|
    if /.msi\z/ =~ package
      next unless msi_enabled?
    end
    puts("Package name: #{package}")
    puts("Uploading...")
    client.upload_asset(url, package)
  end
end
