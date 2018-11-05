require 'active_record_migrations'
require 'mechanize'

ActiveRecordMigrations.load_tasks

desc "Parse"
task :parse => :environment do
  agent = Mechanize.new

  agent.get('http://webprod1.isbe.net/ILEARN/')
  agent.page.forms_with(action: '/ILEARN/Content/SearchData')[0].submit

  (2..87).each do |p|
    # --------------------------------------------------------
    agent.page.links_with(href: /(RCDTSeclected)/).each do |l|
      tables = l.click

      info = tables.search('#DistrictInfo')
      district_number = info.text[/(\d{5,})/].to_i
      district_name = info.text[/- .{1,}$/]

      info = tables.search('.col-md-offset-4 p')
      superintendent = info[0].text[/:.{1,}$/]
      address = info[1].text[/:.{1,}$/]
      phone = info[2].text[/:.{1,}$/]
      district_type = info[3].text[/:.{1,}$/]
      data_source_url = tables.uri
      scrape_dev = 'ce'

      sql = "INSERT INTO ilearn_districts (district_number, district_name, superintendent, address, phone, district_type, data_source_url, scrape_dev) VALUES (#{district_number}, '#{district_name}', '#{superintendent}', '#{address}', '#{phone}', '#{district_type}', '#{data_source_url}', '#{scrape_dev}');"
      ActiveRecord::Base.connection.execute(sql)

      info = tables.search('.table .table-condensed .table-responsive .table-striped .text-center tbody tr td')
      total = tables.search('.table .table-condensed .table-responsive .table-striped .text-center tbody tr td b')

      state_amount = info[1]
      local_amount = info[5]
      federal_amount = info[9]
      total_amount_receint = total[0]

      # TODO 'Добавить district_id'
      sql = "INSERT INTO ilearn_receits_revenues (district_number, district_name, state_amount, local_amount, federal_amount, total_amount, data_source_url, scrape_dev, ilearn_districts_id) VALUES (#{district_number}, '#{district_name}', #{state_amount}, #{local_amount}, #{federal_amount}, #{total_amount_receint}, '#{data_source_url}', '#{scrape_dev}' );"
      ActiveRecord::Base.connection.execute(sql)

      instruction_amount = info[16]
      general_administration_amount = info[20]
      support_services_amount = info[24]
      other_amount = info[28]
      total_amount_exp = total[1]

      eigth_month = info[31]
      statewide_ADA= info[33]
      net_operating= info[37]
      operating_expance = info[39]
      statewide_OEPP= info[43]
      statewide_OEPP= info[41]
      allowance= info[45]
      per_capita= info[47]
      statewide_PCTC = info[49]
      statewide_PCTC= info[51]

      real_eav= info[53]
      real_per_pupil = info[55]
      statewide_eavpp_rank = info[57]
      formula_type = info[59]
      total_tax_rate = info[61]
      statewide_ttr_rank = info[63]
      operating_tax_rate = info[65]
      statewide_otr_rank = info[67]


      puts tables.uri
    end
    # ------------------------------------------------------------
    agent.get("http://webprod1.isbe.net/ILEARN/Content/SearchData?page=#{p}&amp;RCA=1")
    puts agent.page.uri
  end
end