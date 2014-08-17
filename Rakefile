require 'fileutils'

BUILD_DIR = 'build'
DB_FILE = File.join(BUILD_DIR,'muff1nman.db.tar.gz')
IGNORE = ['build']

def add_package_to_db pkg
  puts "Adding #{pkg} to #{DB_FILE}"
  `repo-add --key F8364EDF --sign --verify #{DB_FILE} #{pkg}`
end

def built_pkg_file dir
  puts "Finding built pkg in #{dir}"
  pkgs = Dir.glob("#{dir}/*.pkg.tar.xz")
  puts "The list #{pkgs}"
  return nil unless pkgs.size == 1
  pkgs[0]
end

def ensure_build_dir_present
  FileUtils.mkdir BUILD_DIR unless File.directory? BUILD_DIR
end

def export_package dir
  pkg = built_pkg_file dir
  new_pkg = File.join(BUILD_DIR,File.basename(pkg))
  puts "Exporting #{pkg} to #{new_pkg}"
  ensure_build_dir_present
  FileUtils.cp(pkg,new_pkg)
  add_package_to_db new_pkg
end

def build_package dir
  puts "Building #{dir}"
  Dir.chdir dir do
    `makepkg --key F8364EDF --sign`
  end
end

def clean_package dir
  puts "Finding old packages in dir #{dir}"
  old_pkg = built_pkg_file dir
  unless old_pkg.nil?
    puts "Removing old package #{old_pkg}"
    FileUtils.rm(old_pkg)
  end
end

task :build do
  puts "Building all packages"
  Dir.glob('*').keep_if { |entry| File.directory? entry and !IGNORE.include? entry }.each do |dir|
    clean_package dir
    build_package dir
    export_package dir
  end
end

task :updatedb => [:cleandb] do

end

task :cleandb do
  FileUtils.rm_f(DB_FILE)
end
