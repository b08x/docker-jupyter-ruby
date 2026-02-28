# frozen_string_literal: true

OWNER = (ENV['DOCKER_USER'] || ENV['USER']).freeze
ALL_IMAGES = %w[
  base
  nlp
].each(&:freeze).freeze

BASE_IMAGES = ALL_IMAGES.map do |name|
  base_image_name, base_image_tag = nil
  IO.foreach("#{name}/Containerfile") do |line|
    break if base_image_name && base_image_tag

    case line
    when /BASE_IMAGE_TAG=(\h+)/
      base_image_tag = Regexp.last_match(1)
    when /BASE_IMAGE_TAG=latest/
      base_image_tag = 'latest'
    when /\AFROM\s+([^:]+)/
      base_image_name = Regexp.last_match(1)
    end
  end
  [
    name,
    [base_image_name, base_image_tag].join(':')
  ]
end.to_h

PODMAN_FLAGS = ENV['PODMAN_FLAGS'] || ENV['DOCKER_FLAGS']

TAG_LENGTH = 12

def git_revision
  `git rev-parse HEAD`.chomp
end

def tag_from_commit_sha1
  git_revision[...TAG_LENGTH]
end

ALL_IMAGES.each do |image|
  revision_tag = tag_from_commit_sha1

  desc "Pull the base image for #{OWNER}/#{image} image"
  task "pull/base_image/#{image}" do
    base_image = BASE_IMAGES[image]
    sh "podman pull #{base_image}"
  end

  desc "Build #{OWNER}/#{image} image"
  task "build/#{image}" => "pull/base_image/#{image}" do
    sh "podman build --format docker -f #{image}/Containerfile #{PODMAN_FLAGS} --rm -t #{OWNER}/notebook-#{image}:latest ."
  end

  desc "Make #{OWNER}/#{image} image"
  task "make/#{image}" do
    sh "podman build --format docker --build-arg REGISTRY=localhost --build-arg OWNER=#{OWNER} -f #{image}/Containerfile #{PODMAN_FLAGS} --rm -t #{OWNER}/notebook-#{image}:latest ."
  end

  desc "Tag #{OWNER}/#{image} image"
  task "tag/#{image}" => "build/#{image}" do
    sh "podman tag #{OWNER}/notebook-#{image}:latest #{OWNER}/notebook-#{image}:#{revision_tag}"
  end

  desc "Push #{OWNER}/#{image} image"
  task "push/#{image}" => "tag/#{image}" do
    sh "podman push #{OWNER}/notebook-#{image}:latest"
    sh "podman push #{OWNER}/notebook-#{image}:#{revision_tag}"
  end
end

desc 'Build all images'
task 'build-all' do
  ALL_IMAGES.each do |image|
    Rake::Task["build/#{image}"].invoke
  end
end

desc 'Tag all images'
task 'tag-all' do
  ALL_IMAGES.each do |image|
    Rake::Task["tag/#{image}"].invoke
  end
end

desc 'Push all images'
task 'push-all' do
  ALL_IMAGES.each do |image|
    Rake::Task["push/#{image}"].invoke
  end
end
