# frozen_string_literal: true

fastlane_version '2.181.0'

default_platform :android

version = ''

commitMessage = ''

platform :android do
  before_all do

    reset_git_repo( force: true, files: ['productionVersion.properties','productionErrorVersion.properties'])

    # Get Last commit
    commit = last_git_commit
    keyWordFound = ''
    commitMessage = commit[:message]
    puts "Last git commit : #{commitMessage}"
    matchdata = commitMessage.match(/\[(Major|Minor|Patch|Build*?)\]/i)
    if matchdata.nil?
      puts 'Key word not Found'
      keyWordFound = 'Build'
    else
      matchdata.captures
      puts matchdata[1]
      keyWordFound = matchdata[1]
    end

    case keyWordFound
    when 'Major', 'major'
      version = 'Major'
    when 'Minor', 'minor'
      version = 'Minor'
    when 'Patch', 'patch'
      version = 'Patch'
    else
      version = 'Build'
    end

    puts "Found Version Bump From Last Commit: #{version}"

  end

  desc 'Generate Build for Production Variant and Deploy it on Internal Testing Track'
  lane :internal do |options|
    ensure_git_branch(branch: 'master')
    task_properties = {
      'buildVariantType' => 'production',
      'versionType' => version
    }
    gradle(task: "clean")
    gradle(task: "performVersionCodeAndVersionNumberIncrement",properties: task_properties)
    #gradle(task: "assembleProductionStagingRelease")
    #upload_to_play_store(track: 'internal', release_status: 'draft')
  end

  # This block is called, only if the executed lane was successful
  after_all do |lane, options|
    fileCommitArray = Array.new(2)
    fileCommitArray = ['productionErrorVersion.properties', 'productionVersion.properties']
    messageForCommit = "[ci-skip] Version Bump For Production Variant"
    git_commit(path: fileCommitArray, message: messageForCommit)
    # remote: "origin",         # optional, default: "origin"
    push_to_git_remote(
      local_branch: 'master', # optional, aliased by "branch", default: "master"
      remote_branch: 'master', # optional, default is set to local_branch
      force: true, # optional, default: false
      tags: false # optional, default: true
    )
  end

  error do |lane, exception|
    puts exception.message
    fileArray = Array.new(2)
    fileArray = ['productionErrorVersion.properties', 'productionVersion.properties']
    reset_git_repo(force: true, files: fileArray)
  end
end
