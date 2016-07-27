require 'pathname'
require 'open3'

def log_message(message)
  print("#{message}... ")

  begin
    yield

    puts 'OK'
  rescue StandardError => e
    puts 'FAIL'
  end
end

PROJECT_PATH = Pathname.new(__FILE__).dirname
PROJECT_VERSION = PROJECT_PATH.join('VERSION').read.strip

namespace :build do

  task :readme do
    template_path = PROJECT_PATH.join(*%w[templates Readme.txt])
    template_data = template_path.read

    readme_data = template_data % { version: PROJECT_VERSION }
    readme_path = PROJECT_PATH.join('Readme.md')

    log_message "Generating #{readme_path.relative_path_from(PROJECT_PATH)}" do
      readme_path.open('w+') { |file| file.puts(readme_data) }
    end
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

    log_message "Adding header to #{source_path.relative_path_from(PROJECT_PATH)}" do
      source_path.open('w+') { |file| file.puts(source_data) }
    end
  end

  task default: [:readme, :header]
end

desc 'Build the project'
task build: 'build:default'

task release: :build do
  current_tag = nil

  log_message 'Getting current Git tag' do
    output, error, status = Open3.capture3('git describe --tags') # Use capture3 to capture stderr
    output = output.strip

    current_tag = output unless output.empty?
  end

  if current_tag != PROJECT_VERSION
    log_message('Comitting to Git') { sh "git commit -am '#{PROJECT_VERSION}'" }
    log_message("Adding Git tag #{PROJECT_VERSION}") { sh "git tag #{PROJECT_VERSION}" }
    log_message("Pushing Git tag #{PROJECT_VERSION}") { sh "git push origin #{PROJECT_VERSION}" }
  else
    puts 'ERROR: Git tag and project version are the same'
  end
end
