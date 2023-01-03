# frozen_string_literal: true

RUBY_WASM_PACKAGE = "head-wasm32-unknown-wasi-full-js"
RUBY_WASM_PACKAGE_FILENAME = "ruby-#{RUBY_WASM_PACKAGE}.tar.gz"
RUBY_WASM_VERSION = "2022-12-25-a"
RUREMA_WASM_WORKDIR = File.absolute_path("tmp/#{RUBY_WASM_VERSION}", __dir__)
RUBY_WASM_URL = "https://github.com/ruby/ruby.wasm/releases/download/#{RUBY_WASM_VERSION}/#{RUBY_WASM_PACKAGE_FILENAME}"
BITCLUST_PATH = File.absolute_path('bitclust', __dir__)
DUMMY_LIB_PATH = File.absolute_path('lib', __dir__)
RUREMA_WASM_PATH = File.absolute_path("rurema.wasm", __dir__)

directory RUREMA_WASM_WORKDIR

file "#{RUREMA_WASM_WORKDIR}/#{RUBY_WASM_PACKAGE_FILENAME}" do
  Dir.chdir(RUREMA_WASM_WORKDIR) do
    sh "curl", "-sLO", RUBY_WASM_URL
  end
end

directory "#{RUREMA_WASM_WORKDIR}/#{RUBY_WASM_PACKAGE}" => "#{RUREMA_WASM_WORKDIR}/#{RUBY_WASM_PACKAGE_FILENAME}" do
  Dir.chdir(RUREMA_WASM_WORKDIR) do
    sh "tar", "xfz", RUBY_WASM_PACKAGE_FILENAME
  end
end

file "#{RUREMA_WASM_WORKDIR}/ruby.wasm" => "#{RUREMA_WASM_WORKDIR}/#{RUBY_WASM_PACKAGE}" do
  Dir.chdir(RUREMA_WASM_WORKDIR) do
    ruby_path = File.join(RUBY_WASM_PACKAGE, 'usr/local/bin/ruby')
    sh "cp", ruby_path, "ruby.wasm"
  end
end

directory "#{RUREMA_WASM_WORKDIR}/bitclust" do
  Dir.chdir('bitclust') do
    sh "git", "archive", "--format", "zip", "--output", "#{RUREMA_WASM_WORKDIR}/bitclust.zip", "HEAD"
  end
  Dir.chdir(RUREMA_WASM_WORKDIR) do
    sh "unzip", "-od", "bitclust", "bitclust.zip"
  end
end

task :generate do
  Dir.chdir('doctree') do
    sh 'rake', 'generate'
  end
end

desc "build bitclust wasm"
task build_rurema_wasm: [:generate, "#{RUREMA_WASM_WORKDIR}/bitclust", "#{RUREMA_WASM_WORKDIR}/ruby.wasm"] do
  database_maps = Dir['/tmp/db-*/properties'].flat_map do |properties_path|
    database = File.basename(File.dirname(properties_path))
    ["--mapdir", "/#{database}::/tmp/#{database}"]
  end
  p database_maps
  Dir.chdir(RUREMA_WASM_WORKDIR) do
    sh(
      "wasi-vfs",
      "pack",
      "ruby.wasm",
      "--mapdir",
      "/dummy::#{DUMMY_LIB_PATH}",
      "--mapdir",
      "/usr::./#{RUBY_WASM_PACKAGE}/usr",
      "--mapdir",
      "/bitclust::./bitclust",
      *database_maps,
      "--output",
      RUREMA_WASM_PATH
    )
  end
end

task default: :build_rurema_wasm
