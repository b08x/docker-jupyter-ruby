OWNER = 'b08x'.freeze
IMAGES = {
  'base' => %w[cpu gpu],
  'nlp' => %w[cpu gpu]
}.freeze

DOCKER_FLAGS = ENV['DOCKER_FLAGS']
TAG_LENGTH = 12

def git_revision
  `git rev-parse HEAD`.chomp
end

def tag_from_commit_sha1
  git_revision[...TAG_LENGTH]
end

def get_base_image(dockerfile_path)
  args = {}
  from_line = nil

  IO.foreach(dockerfile_path) do |line|
    case line
    when /^\s*ARG\s+([^=]+)=(.+)/
      args[Regexp.last_match(1).strip] = Regexp.last_match(2).strip
    when /^\s*FROM\s+(.+)/
      from_line = Regexp.last_match(1).strip
      break
    end
  end

  return nil unless from_line

  # Substitute ARG variables
  from_line.gsub!(/\$\{?(\w+)\}?/) do |match|
    args.fetch(Regexp.last_match(1), match)
  end

  from_line
end

IMAGES.each do |image, variants|
  variants.each do |variant|
    dockerfile = variant == 'gpu' ? "#{image}/Dockerfile.gpu" : "#{image}/Dockerfile"
    next unless File.exist?(dockerfile)

    image_tag = "#{OWNER}/notebook-#{image}:#{variant}"
    revision_tag = "#{OWNER}/notebook-#{image}:#{variant}-#{tag_from_commit_sha1}"

    dependencies = []
    dependencies << "build/base/#{variant}" if image == 'nlp'

    desc "Build #{image_tag}"
    task "build/#{image}/#{variant}" => dependencies do
      if image == 'base'
        base_image = get_base_image(dockerfile)
        puts "Pulling base image for #{image}/#{variant}: #{base_image}"
        sh "docker pull #{base_image}"
      end
      sh "docker build -f #{dockerfile} #{DOCKER_FLAGS} --rm --force-rm -t #{image_tag} ."
    end

    desc "Make #{image_tag} (force build without cache)"
    task "make/#{image}/#{variant}" => dependencies do
      if image == 'base'
        base_image = get_base_image(dockerfile)
        puts "Pulling base image for #{image}/#{variant}: #{base_image}"
        sh "docker pull #{base_image}"
      end
      sh "docker build -f #{dockerfile} #{DOCKER_FLAGS} --no-cache --rm --force-rm -t #{image_tag} ."
    end

    desc "Tag #{image_tag} with git revision"
    task "tag/#{image}/#{variant}" => "build/#{image}/#{variant}" do
      sh "docker tag #{image_tag} #{revision_tag}"
    end

    desc "Push #{image_tag} and revision tag"
    task "push/#{image}/#{variant}" => "tag/#{image}/#{variant}" do
      sh "docker push #{image_tag}"
      sh "docker push #{revision_tag}"
    end
  end

  desc "Build all variants for #{image}"
  task "build/#{image}" do
    variants.each do |variant|
      if File.exist?(variant == 'gpu' ? "#{image}/Dockerfile.gpu" : "#{image}/Dockerfile")
        Rake::Task["build/#{image}/#{variant}"].invoke
      end
    end
  end

  desc "Tag all variants for #{image}"
  task "tag/#{image}" do
    variants.each do |variant|
      if File.exist?(variant == 'gpu' ? "#{image}/Dockerfile.gpu" : "#{image}/Dockerfile")
        Rake::Task["tag/#{image}/#{variant}"].invoke
      end
    end
  end

  desc "Push all variants for #{image}"
  task "push/#{image}" do
    variants.each do |variant|
      if File.exist?(variant == 'gpu' ? "#{image}/Dockerfile.gpu" : "#{image}/Dockerfile")
        Rake::Task["push/#{image}/#{variant}"].invoke
      end
    end
  end
end

desc 'Build all images and variants'
task 'build-all' do
  IMAGES.each_key do |image|
    Rake::Task["build/#{image}"].invoke
  end
end

desc 'Tag all images and variants'
task 'tag-all' do
  IMAGES.each_key do |image|
    Rake::Task["tag/#{image}"].invoke
  end
end

desc 'Push all images and variants'
task 'push-all' do
  IMAGES.each_key do |image|
    Rake::Task["push/#{image}"].invoke
  end
end

task default: 'build-all'
