require 'octokit'

Octokit.auto_paginate = true
source_client = if ENV['SOURCE_API_ENDPOINT']
  Octokit::Client.new :access_token => ENV['SOURCE_GITHUB_ACCESS_TOKEN'],
                      :api_endpoint => ENV['SOURCE_API_ENDPOINT']
else
  Octokit::Client.new :access_token => ENV['GITHUB_ACCESS_TOKEN']
end
target_client = if ENV['TARGET_API_ENDPOINT']
  Octokit::Client.new :access_token => ENV['TARGET_GITHUB_ACCESS_TOKEN'],
                      :api_endpoint => ENV['TARGET_API_ENDPOINT']
else
  source_client
end

# Array[members_to_add Array, members_to_remove Array, unchanged Array]
def reconcile_members(source, target)
  [
    source - target,
    target - source,
    source & target
  ]
end

namespace :team do
  task :sync do
    source_org, source_team_name = ENV['SOURCE'].split('/')
    target_org, target_team_name = ENV['TARGET'].split('/')

    source_org_teams = source_client.organization_teams(source_org)

    target_org_teams =
      if source_org == target_org
        source_org_teams
      else
        target_client.organization_teams(target_org)
      end

    source_id = source_org_teams.detect{ |t| t.slug == source_team_name }[:id]

    target_team = target_org_teams.detect{ |t| t.slug == target_team_name }
    target_id = if target_team
      target_team.id
    else
      target_client.create_team(target_org, {:name => target_team_name}).id
    end

    # https://developer.github.com/v3/orgs/teams/#list-team-members
    source_members = Set.new source_client.team_members(source_id).map(&:login)
    target_members = Set.new target_client.team_members(target_id).map(&:login)

    to_add, to_remove, rest = reconcile_members(source_members, target_members)

    to_add.each do |login|
      # https://developer.github.com/v3/orgs/teams/#add-team-member
      target_client.add_team_member(target_id, login)
    end

    to_remove.each do |login|
      # https://developer.github.com/v3/orgs/teams/#remove-team-member
      target_client.remove_team_member(target_id, login)
    end
  end
end

task :default => "team:sync"
