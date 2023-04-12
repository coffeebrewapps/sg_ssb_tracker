# frozen_string_literal: true

require "http"
require "uri"
require "date"
require "json"
require "fileutils"

desc "Create a success scenario test files"
task :fetch_issues, [:issues_url] do |_t, args|
  issues_url = args[:issues_url]

  earliest_year = 2015
  current_year = Date.today.year

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

  FileUtils.mkdir_p("data")

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
end
