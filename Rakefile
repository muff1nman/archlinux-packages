require 'fileutils'
require 'kramdown'

BUILD_DIR = 'build'
DB_FILE = File.join(BUILD_DIR,'muff1nman.db.tar.gz')
IGNORE = ['build' 'thirdparty']

def add_package_to_db pkg
  puts "Adding #{pkg} to #{DB_FILE}"
  exit unless system "repo-add -f --key F8364EDF --sign #{DB_FILE} #{pkg}"
end

def built_pkg_files dir
  Dir.glob("#{dir}/*.pkg.tar.xz")
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
    FileUtils.cp("#{pkg}.sig","#{new_pkg}.sig")
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
    exit unless system "makepkg -sr --key F8364EDF --sign"
  end
end

def clean_package dir
  puts "Finding old packages in dir #{dir}"
  do_with_pkgs_in(dir,false) do |old_pkg|
    puts "Removing old package #{old_pkg}"
    FileUtils.rm_f(old_pkg)
    FileUtils.rm_f("#{old_pkg}.sig")
  end
end

task :build, :pkg_list do |t,args|
  buildpkgs args[:pkg_list]
end

def buildpkgs pkg_list="*"
  pkg_list = "*" if pkg_list.nil?
  if pkg_list == "*"
    puts "Building all packages"
  else
    puts "Building #{pkg_list}"
  end
  prep_build_dir
  Dir.glob(pkg_list).keep_if { |entry| File.directory? entry and !IGNORE.include? entry }.each do |dir|
    build_package dir
    export_package dir
  end
end

task :index do
  Dir.chdir BUILD_DIR do
    markdown_content = Dir
      .glob('*')
      .reject { |item| item.eql? "index.html" }
      .sort
      .map { |item| "[#{item}](#{item})" }
      .join "\n\n"
    index_content = Kramdown::Document.new(markdown_content).to_html
    File.open('index.html', 'w') { |file| file.write index_content }
  end
end

task :download do
  prep_build_dir
end

def prep_build_dir
  ensure_build_dir_present
  Dir.chdir BUILD_DIR do
    exit unless system "s3cmd get -r -F --check-md5 --skip-existing s3://repo.andrewdemaria.com/archlinux/ ."
  end
end

task :upload do
  Dir.chdir BUILD_DIR do
    exec "s3cmd sync -r -F --delete-removed . s3://repo.andrewdemaria.com/archlinux/"
  end
end

task :cleandb do
  puts "Removing database file"
  FileUtils.rm_f(DB_FILE)
end

task :clean do
  exit unless system "git clean -fxd"
end
