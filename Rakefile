require 'active_record_migrations'
require 'mechanize'
require 'date'
require 'json'


ActiveRecordMigrations.load_tasks

desc "Parse"
task :parse => :environment do

  agent = Mechanize.new

  page = agent.get('http://webprod1.isbe.net/ILEARN/Content/SearchData?page=1&RCA=1')

  [1..86].each do |list|
    school_links = page.links_with('href: %r{ RCDTSeclected }')
    school_links.each do |link|
      view = link.click
      view = view.search('#DistrictInfo')
      view = view.search('.col-md-offset-4 p')
    end
  end
end
