require 'fileutils'

BUILD_DIR = 'build'
DB_FILE = File.join(BUILD_DIR,'muff1nman.db.tar.gz')
IGNORE = ['build']

def add_package_to_db pkg
  puts "Adding #{pkg} to #{DB_FILE}"
  `repo-add -f --key F8364EDF --sign #{DB_FILE} #{pkg}`
end

def built_pkg_files dir
  puts "Finding built pkg in #{dir}"
  pkgs = Dir.glob("#{dir}/*.pkg.tar.xz") + Dir.glob("{dir}/*.pkg.tar.xz.sig")
  puts "The list #{pkgs}"
  pkgs
end

def ensure_build_dir_present
  FileUtils.mkdir BUILD_DIR unless File.directory? BUILD_DIR
end

def export_package dir
  ensure_build_dir_present
  do_with_pkgs_in(dir,true) do |pkg|
    new_pkg = File.join(BUILD_DIR,File.basename(pkg))
    puts "Exporting #{pkg} to #{new_pkg}"
    FileUtils.cp(pkg,new_pkg)
    add_package_to_db new_pkg
  end
end

def do_with_pkgs_in dir, abort_on_error, &block
  do_with_pkg_list( built_pkg_files(dir), abort_on_error, &block )
end

def do_with_pkg_list pkg_list, abort_on_error, &block
  case pkg_list
  when Array
    case pkg_list.size
    when 0
      # error case to be handled later...
    else
      pkg_list.each{|pkg| yield pkg }
      return
    end
  when String
    yield pkg_list
    return
  end
  abort "Cannot operate on #{pkg_list} as package list" if abort_on_error
end

def build_package dir
  puts "Building #{dir}"
  Dir.chdir dir do
    `makepkg -sr --key F8364EDF --sign`
  end
end

def clean_package dir
  puts "Finding old packages in dir #{dir}"
  do_with_pkgs_in(dir,false) do |old_pkg|
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

task :upload => :build do
  Dir.chdir BUILD_DIR do
    `s3cmd sync . s3://repo.andrewdemaria.com/archlinux/`
  end
end

task :cleandb do
  FileUtils.rm_f(DB_FILE)
end
