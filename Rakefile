require 'octokit'

Octokit.auto_paginate = true
client = Octokit::Client.new :access_token => ENV['GITHUB_ACCESS_TOKEN']

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
    source_org, source_team = ENV['SOURCE'].split('/')
    target_org, target_team = ENV['TARGET'].split('/')

    source_org_teams = client.organization_teams(source_org)
    target_org_teams =
      if source_org == target_org
        source_org_teams
      else
        client.organization_teams(target_org)
      end

    source_id = source_org_teams.detect{ |t| t.slug == source_team }[:id]
    target_id = source_org_teams.detect{ |t| t.slug == target_team }[:id]

    # https://developer.github.com/v3/orgs/teams/#list-team-members
    source_members = Set.new client.team_members(source_id).map(&:login)
    target_members = Set.new client.team_members(target_id).map(&:login)

    to_add, to_remove, rest = reconcile_members(source_members, target_members)

    to_add.each do |login|
      # https://developer.github.com/v3/orgs/teams/#add-team-member
      client.add_team_member(target_id, login)
    end

    to_remove.each do |login|
      # https://developer.github.com/v3/orgs/teams/#remove-team-member
      client.remove_team_member(target_id, login)
    end
  end
end

task :default => "team:sync"
