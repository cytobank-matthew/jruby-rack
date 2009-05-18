#--
# Copyright 2007-2009 Sun Microsystems, Inc.
# This source code is available under the MIT license.
# See the file LICENSE.txt for details.
#++

raise "JRuby-Rack must be built with JRuby: try again with `jruby -S rake'" unless defined?(JRUBY_VERSION)

require 'rake/clean'

def compile_classpath
  if ENV['JRUBY_PARENT_CLASSPATH']
    classpath = []
    ENV['JRUBY_PARENT_CLASSPATH'].split(File::PATH_SEPARATOR).each {|p| classpath << p}
  else
    java_classpath = Java::JavaLang::System.getProperty("java.class.path").split(File::PATH_SEPARATOR)
    classpath = Dir["#{File.expand_path 'src/main/lib'}/*.jar"] + java_classpath
  end
end

def test_classpath
  compile_classpath + [File.expand_path("target/classes"), File.expand_path("target/test-classes")]
end

CLEAN << 'target'

directory 'target/classes'

desc "Compile java classes"
task :compile => "target/classes" do |t|
  sh "javac -classpath #{compile_classpath.join(File::PATH_SEPARATOR)} -source 1.5 " +
    "-target 1.5 -d #{t.prerequisites.first} #{Dir['src/main/java/**/*.java'].join(' ')}"
end

directory 'target/test-classes'

desc "Compile classes used for test/spec"
task :compilespec => "target/test-classes" do |t|
  sh "javac -classpath #{test_classpath.join(File::PATH_SEPARATOR)} -source 1.5 " +
    "-target 1.5 -d #{t.prerequisites.first} #{Dir['src/spec/java/**/*.java'].join(' ')}"
end

desc "Unpack the rack gem"
task :unpack_gem => "target" do |t|
  Dir.chdir(t.prerequisites.first) do
    unless File.directory?("rack")
      ruby "-S", "gem", "unpack", FileList["../src/main/lib/rack*.gem"].first
      mv FileList["rack-*"].first, "rack"
    end
  end
end

version_file = 'src/main/ruby/jruby/rack/version.rb'
load version_file

task :update_version do
  if ENV["VERSION"] && ENV["VERSION"] != JRuby::Rack::VERSION
    lines = File.readlines(version_file)
    lines.each {|l| l.sub!(/VERSION =.*$/, %{VERSION = "#{ENV["VERSION"]}"})}
    File.open(version_file, "wb") {|f| f.puts *lines }
  end
end


task :test_resources => ["target/test-classes"] do |t|
  FileList["src/spec/ruby/merb/gems/gems/merb-core-*/lib/*"].each do |f|
    cp_r f, t.prerequisites.first
  end
end

desc "Copy resources"
task :resources => ["target/classes", :unpack_gem, :update_version, :test_resources] do |t|
  ['src/main/ruby', 'target/rack/lib'].each do |dir|
    FileList["#{dir}/*"].each do |f|
      cp_r f, t.prerequisites.first
    end
  end
  meta_inf = File.join(t.prerequisites.first, "META-INF")
  mkdir_p meta_inf
  cp "src/main/tld/jruby-rack.tld", meta_inf
end

task :speconly do
  if ENV['SKIP_SPECS'] && ENV['SKIP_SPECS'] == "true"
    puts "Skipping specs due to SKIP_SPECS=#{ENV['SKIP_SPECS']}"
  else
    test_classpath.each {|p| $CLASSPATH << p }
    opts = ["--format", "specdoc"]
    opts << ENV['SPEC_OPTS'] if ENV['SPEC_OPTS']
    spec = ENV['SPEC'] || "src/spec/ruby/**/*_spec.rb"
    opts.push *FileList[spec].to_a
    ENV['CLASSPATH'] = test_classpath.join(File::PATH_SEPARATOR)
    ruby "-S", "spec", *opts
  end
end

desc "Run specs"
task :spec => [:compile, :resources, :compilespec, :speconly]

task :test => :spec

file "target/jruby-rack-#{JRuby::Rack::VERSION}.jar" => :spec do |t|
  sh "jar cf #{t.name} -C target/classes ."
end

desc "Create the jar"
task :jar => "target/jruby-rack-#{JRuby::Rack::VERSION}.jar"

task :default => :jar

task :install => "target/jruby-rack-#{JRuby::Rack::VERSION}.jar" do |t|
  repos_dir = File.expand_path "~/.m2/repository/org/jruby/rack/jruby-rack/#{JRuby::Rack::VERSION}"
  mkdir_p repos_dir
  cp t.prerequisites.first, repos_dir
  cp "pom.xml", "#{repos_dir}/jruby-rack-#{JRuby::Rack::VERSION}.pom"
end

task :classpaths do
  puts "compile_classpath:",*compile_classpath
  puts "test_classpath:", *test_classpath
end


require 'rubygems' 
gem 'ci_reporter' 
require 'ci/reporter/rake/test_unit' 