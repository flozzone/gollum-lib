require 'rubygems'
require 'rake'
require 'date'

#############################################################################
#
# Helper functions
#
#############################################################################

def name
  "gollum-lib"
end

def version
  line = File.read("lib/gollum-lib/version.rb")[/^\s*VERSION\s*=\s*.*/]
  line.match(/.*VERSION\s*=\s*['"](.*)['"]/)[1]
end

# assumes x.y.z all digit version
def next_version
  # x.y.z
  v = version.split '.'
  # bump z
  v[-1] = v[-1].to_i + 1
  v.join '.'
end

def bump_version
  old_file = File.read("lib/gollum-lib/version.rb")
  old_version_line = old_file[/^\s*VERSION\s*=\s*.*/]
  new_version = next_version
  # replace first match of old version with new version
  old_file.sub!(old_version_line, "    VERSION = '#{new_version}'")

  File.write("lib/gollum-lib/version.rb", old_file)

  new_version
end

def date
  Date.today.to_s
end

def rubyforge_project
  name
end

def gemspec_file
  "gemspec.rb"
end

def gemspecs
  ["#{name}.gemspec", "#{name}_java.gemspec"]
end

def gem_files
  ["#{name}-#{version}.gem", "#{name}-#{version}-java.gem"]
end

def replace_header(head, header_name)
  head.sub!(/(\.#{header_name}\s*= ').*'/) { "#{$1}#{send(header_name)}'"}
end

#############################################################################
#
# Standard tasks
#
#############################################################################

task :default => :test

require 'rake/testtask'
Rake::TestTask.new(:test) do |test|
  test.libs << 'lib' << 'test' << '.'
  test.pattern = 'test/**/test_*.rb'
  test.verbose = true
end

desc "Generate RCov test coverage and open in your browser"
task :coverage do
  require 'rcov'
  sh "rm -fr coverage"
  sh "rcov test/test_*.rb"
  sh "open coverage/index.html"
end

desc "Open an irb session preloaded with this library"
task :console do
  sh "irb -rubygems -r ./lib/#{name}.rb"
end

#############################################################################
#
# Custom tasks (add your own tasks here)
#
#############################################################################

desc "Update version number and gemspec"
task :bump do
  puts "Updated version to #{bump_version}"
  # Execute does not invoke dependencies.
  # Manually invoke gemspec then validate.
  Rake::Task[:gemspec].execute
  Rake::Task[:validate].execute
end

desc "Build and install"
task :install => :build do
  sh "gem install --local --no-ri --no-rdoc pkg/#{name}-#{version}.gem"
end

#############################################################################
#
# Packaging tasks
#
#############################################################################

desc 'Create a release build'
task :release => :build do
  unless `git branch` =~ /^\* master$/
    puts "You must be on the master branch to release!"
    exit!
  end
  sh "git commit --allow-empty -a -m 'Release #{version}'"
  sh "git pull --rebase origin master"
  sh "git tag v#{version}"
  sh "git push origin master"
  sh "git push origin v#{version}"
  sh "gem push pkg/#{name}-#{version}.gem"
  sh "gem push pkg/#{name}-#{version}-java.gem"
end

desc 'Publish to rubygems. Same as release'
task :publish => :release

desc 'Build gem'
task :build => :gemspec do
  sh "mkdir -p pkg"
  gemspecs.each do |gemspec|
    sh "gem build #{gemspec}"
  end
  gem_files.each do |gem_file|
    sh "mv #{gem_file} pkg"
  end
end

desc 'Update gemspec'
task :gemspec => :validate do
  # read spec file and split out manifest section
  spec = File.read(gemspec_file)
  head, manifest, tail = spec.split("  # = MANIFEST =\n")

  # replace name version and date
  replace_header(head, :name)
  replace_header(head, :date)
  #comment this out if your rubyforge_project has a different name
  replace_header(head, :rubyforge_project)

  # determine file list from git ls-files
  files = `git ls-files`.
    split("\n").
    sort.
    reject { |file| file =~ /^\./ }.
    reject { |file| file =~ /^(rdoc|pkg|test|Home\.md|\.gitattributes|Guardfile)/ }.
    map { |file| "    #{file}" }.
    join("\n")

  # piece file back together and write
  manifest = "  s.files = %w(\n#{files}\n  )\n"
  spec = [head, manifest, tail].join("  # = MANIFEST =\n")
  File.open(gemspec_file, 'w') { |io| io.write(spec) }
  puts "Updated #{gemspec_file}"
end

desc 'Validate lib files and version file'
task :validate do
  libfiles = Dir['lib/*'] - ["lib/#{name}.rb", "lib/#{name}"]
  unless libfiles.empty?
    puts "Directory `lib` should only contain a `#{name}.rb` file and `#{name}` dir."
    exit!
  end
  unless Dir['VERSION*'].empty?
    puts "A `VERSION` file at root level violates Gem best practices."
    exit!
  end
end
