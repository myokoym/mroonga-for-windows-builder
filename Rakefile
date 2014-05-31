require "fileutils"
require "open-uri"
require "archive/zip"
require "octokit"

mariadb_version = "10.0.11"
mroonga_version = "4.03"
vc_version = "2013"
repo = "mroonga/mroonga"
source_name = "mariadb-#{mariadb_version}-with-mroonga-#{mroonga_version}-for-windows"
build_base_name = "build-vc#{vc_version}"

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
  Archive::Zip.extract("#{source_name}.zip", "source")
end

file "#{build_base_name}.bat" do
  puts("Build scripts: #{build_base_name}*.bat")
  base_url = "https://raw.githubusercontent.com/mroonga/mroonga/master/packages/windows/"
  bat_files = [
    "#{build_base_name}.bat",
    "#{build_base_name}-zip-32.bat",
    "#{build_base_name}-zip-64.bat",
    "#{build_base_name}-msi-32.bat",
    "#{build_base_name}-msi-64.bat",
  ]
  puts("Downloading...")
  bat_files.each do |bat_file|
    url = "#{base_url}#{bat_file}"
    download(url)
  end
end

desc "Build Mroonga for Windows"
task :build => ["source", "#{build_base_name}.bat"] do
  sh("./build-vc#{vc_version}.bat")
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
