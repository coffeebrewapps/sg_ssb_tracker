# frozen_string_literal: true

require "http"
require "uri"
require "date"
require "json"
require "fileutils"

desc "Fetch interests data for issue code"
task :fetch_interests, [:interests_url, :issue_code] do |_t, args|
  interests_url = args[:interests_url]
  issue_code = args[:issue_code]

  if File.exist?("data/interests/#{issue_code}.json")
    puts "#{issue_code} interests already updated, exiting..."
    exit 0
  end

  params = {
    rows: 1,
    filters: "issue_code:#{issue_code}"
  }

  issues_data = []

  response = HTTP.get(interests_url, { params: params })

  parsed_response = response.parse

  if response.status.success? && parsed_response["success"]
    if parsed_response.dig("result", "total").positive?
      records = parsed_response.dig("result", "records")
      issues_data.concat(records)
    else
      puts "No interests record for #{issue_code}, skipping file write..."
      exit 0
    end
  else
    puts "error fetching data: #{response.body}"
  end

  FileUtils.mkdir_p("data/interests")

  File.write(
    "data/interests/#{issue_code}.json",
    JSON.pretty_generate(
      "success" => true,
      "result" => {
        "total" => issues_data.size,
        "records" => issues_data
      }
    )
  )
end

def fetch_issues_interest(issues_data, interests_url)
  issues_data.each do |issue_data|
    issue_code = issue_data["issue_code"]
    system("bundle exec rake fetch_interests[#{interests_url},#{issue_code}]")
  end
end

desc "Fetch issuance data"
task :fetch_issues, [:issues_url, :interests_url] do |_t, args|
  issues_url = args[:issues_url]
  interests_url = args[:interests_url]

  earliest_year = 2015
  current_year = Date.today.year

  FileUtils.mkdir_p("data")

  fetched_latest_year = if File.exist?("data/metadata.json")
                          JSON.parse(File.read("data/metadata.json"))["latest_year"].to_i
                        end

  if fetched_latest_year == current_year
    puts "No new issues to fetch, skipping..."
    issues_data = JSON.parse(File.read("data/issues.json")).dig("result", "records")
  else
    issues_data = []

    (earliest_year..current_year).each do |year|
      start_date = Date.new(year, 1, 1)
      end_date = Date.new(year + 1, 1, 1).prev_day

      params = {
        filters: "issue_type:\"S\" AND ann_date:[#{start_date} TO #{end_date}]",
        sort: "ann_date asc"
      }

      puts "Fetching from #{params}"

      response = HTTP.get(issues_url, { params: params })

      parsed_response = response.parse

      if response.status.success? && parsed_response["success"]
        records = parsed_response.dig("result", "records")
        issues_data.concat(records)
      else
        puts "error fetching data: #{response.body}"
      end
    end

    File.write(
      "data/issues.json",
      JSON.pretty_generate(
        "success" => true,
        "result" => {
          "total" => issues_data.size,
          "records" => issues_data
        }
      )
    )

    File.write(
      "data/metadata.json",
      JSON.pretty_generate(
        "latest_year" => current_year
      )
    )
  end

  fetch_issues_interest(issues_data, interests_url)
end
