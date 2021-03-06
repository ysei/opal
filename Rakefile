require 'bundler'
Bundler.require

require 'opal-sprockets'

desc "Run tests through mspec"
task :default do
  require 'rack'
  require 'webrick'

  Opal::Processor.arity_check_enabled = true

  server = fork do
    serv = Opal::Server.new { |s|
      s.append_path 'spec' # before mspec, so we use our overrides
      s.append_path File.join(Gem::Specification.find_by_name('mspec').gem_dir, 'lib')
      s.debug = false
      s.main = 'ospec/main'
    }

    Rack::Server.start(:app => serv, :Port => 9999, :AccessLog => [],
      :Logger => WEBrick::Log.new("/dev/null"))
  end

  system "phantomjs \"spec/ospec/sprockets.js\" \"http://localhost:9999/\""
  success = $?.success?

  Process.kill(:SIGINT, server)
  Process.wait

  exit 1 unless success
end

desc "Build opal.js and opal-parser.js to build/"
task :dist do
  Opal::Processor.arity_check_enabled = false

  env = Sprockets::Environment.new
  Opal.paths.each { |p| env.append_path p }

  Dir.mkdir 'build' unless File.directory? 'build'

  File.open('build/opal.js', 'w+') { |f| f << env['opal'].to_s }
  File.open('build/opal-parser.js', 'w+') { |f| f << env['opal-parser'].to_s }
end

desc "Check file sizes for opal.js runtime"
task :sizes do
  Opal::Processor.arity_check_enabled = false

  env = Sprockets::Environment.new
  Opal.paths.each { |p| env.append_path p }

  src = env['opal'].to_s
  min = uglify src
  gzp = gzip min

  puts "development: #{src.size}, minified: #{min.size}, gzipped: #{gzp.size}"
end

desc "Rebuild grammar.rb for opal parser"
task :racc do
  %x(racc -l lib/opal/grammar.y -o lib/opal/grammar.rb)
end

# Used for uglifying source to minify
def uglify(str)
  IO.popen('uglifyjs -nc', 'r+') do |i|
    i.puts str
    i.close_write
    return i.read
  end
end

# Gzip code to check file size
def gzip(str)
  IO.popen('gzip -f', 'r+') do |i|
    i.puts str
    i.close_write
    return i.read
  end
end
