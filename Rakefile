require 'pathname'
require 'open3'

PROJECT_PATH = Pathname.new(__FILE__).dirname

def get_current_version
  current_tag, error, status = Open3.capture3('git describe --tags') # Use capture3 to capture stderr
  current_tag = current_tag.strip

  current_tag.empty? ? '0.0.0' : current_tag
end

PROJECT_VERSION = get_current_version

namespace :build do
  
  task :readme do
    template_path = PROJECT_PATH.join(*%w[templates Readme.txt])
    template_data = template_path.read

    readme_data = template_data % { version: PROJECT_VERSION }
    readme_path = PROJECT_PATH.join('Readme.md')

    readme_path.open('w+') { |file| file.puts(readme_data) }
  end

  task :header do
    source_path = PROJECT_PATH.join(*%w[lib dissonance.py])
    source_data = source_path.read

    # Strip current header
    scanning_header_comments = true
    source_data = source_data.lines.each_with_object([]) do |line, memo|
      next if scanning_header_comments && line =~ /^\s*(#.*)?$/
      scanning_header_comments = false

      memo << line
    end.join

    header_path = PROJECT_PATH.join(*%w[templates header.txt])
    header_data = header_path.read
    header_data = header_data % { version: PROJECT_VERSION }

    source_data = "#{header_data}\n#{source_data}"

    source_path.open('w+') { |file| file.puts(source_data) }
  end

  task default: [:readme, :header]
end

desc 'Build the project'
task build: 'build:default'
