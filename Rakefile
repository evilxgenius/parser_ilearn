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

      sql = "INSERT INTO ilearn_districts (district_number, district_name,
      superintendent, address, phone, district_type, data_source_url, scrape_dev)
      VALUES (#{district_number}, '#{district_name}', '#{superintendent}', '#{address}',
      '#{phone}', '#{district_type}', '#{data_source_url}', '#{scrape_dev}');"
      ActiveRecord::Base.connection.execute(sql)

      info = tables.search('.table .table-condensed .table-responsive .table-striped .text-center tbody tr td')
      total = tables.search('.table .table-condensed .table-responsive .table-striped .text-center tbody tr td b')

      state_amount = info[1].text
      local_amount = info[5].text
      federal_amount = info[9].text
      total_amount_receint = total[0].text

      sql = "SELECT id FROM ilearn_districts WHERE district_number = #{district_number}"
      ilearn_district_id = ActiveRecord::Base.connection.execute(sql)

      sql = "INSERT INTO ilearn_receits_revenues (district_number, district_name,
             state_amount, local_amount, federal_amount, total_amount, data_source_url,
             scrape_dev, ilearn_districts_id) VALUES (#{district_number}, '#{district_name}',
             #{state_amount}, #{local_amount}, #{federal_amount}, #{total_amount_receint}, '#{data_source_url}',
             '#{scrape_dev}', #{ilearn_district_id} );"
      ActiveRecord::Base.connection.execute(sql)

      instruction_amount = info[16].text
      general_administration_amount = info[20].text
      support_services_amount = info[24].text
      other_amount = info[28].text
      total_amount_exp = total[1].text

      sql = "INSERT INTO ilearn_expenditures_disbursements (district_number,
            district_name, instruction_amount, general_administration_amount,
            suppert_services_amount, other_amount, total_amount, data_source_url,
            scrape_dev, ilearn_districts_id) VALUES (#{district_number}, '#{district_name}',
            #{instruction_amount}, #{general_administration_amount}, #{support_services_amount},
            #{other_amount}, #{total_amount_exp}, '#{data_source_url}', '#{scrape_dev}', #{ilearn_district_id});"
      ActiveRecord::Base.connection.execute(sql)

      eigth_month = info[31].text
      statewide_ADA= info[33].text
      net_operating= info[37].text
      operating_expance = info[39].text
      statewide_OEPP_rank= info[43].text
      statewide_OEPP= info[41].text
      allowance= info[45].text
      per_capita= info[47].text
      statewide_PCTC_rank = info[49].text
      statewide_PCTC= info[51].text

      sql = "INSERT INTO ilearn_per_student_info (district_number, district_name,
            '9-Month Avg. Daily Attendance', 'Statewide ADA', 'Net Operating Expanse',
            'Operating Expance Per Pupil (OEPP)', 'Statewide OEPP Rank', 'Statewide OEPP',
            'Allowance for Tuition Computation', 'Per Capita Tuition Charge (PCTC)',
            'Statewide PCTC Rank', 'Statewide PCTC', data_source_url, scrape_dev, ilearn_districts_id)
            VALUES (#{district_number}, '#{district_number}', #{eigth_month}, #{statewide_ADA},
            #{net_operating}, #{operating_expance}, #{statewide_OEPP_rank}, #{statewide_OEPP}, #{allowance},
            #{per_capita}, #{statewide_PCTC_rank}, #{statewide_PCTC}, '#{data_source_url}', '#{scrape_dev}',
            #{ilearn_district_id});"
      ActiveRecord::Base.connection.execute(sql)

      real_eav= info[53].text
      real_per_pupil = info[55].text
      statewide_eavpp_rank = info[57].text
      formula_type = info[59].text
      total_tax_rate = info[61].text
      statewide_ttr_rank = info[63].text
      operating_tax_rate = info[65].text
      statewide_otr_rank = info[67].text

      sql = "INSERT INTO ilearn_tax_information (district_number, district_name,
            real_eav, real_per_pupil, statewide_eavpp_rank, formula_type, total_tax_rate,
            statewide_ttr_rank, operating_tax_rate, statewide_otr_rank, ilearn_district_id)
            VALUES (#{district_number}, '#{district_name}', #{real_eav}, #{real_per_pupil},
            #{statewide_eavpp_rank}, '#{formula_type}', #{total_tax_rate}, #{statewide_ttr_rank},
            #{operating_tax_rate}, #{statewide_otr_rank});"
      ActiveRecord::Base.connection.execute(sql)
    end
    # ------------------------------------------------------------
    agent.get("http://webprod1.isbe.net/ILEARN/Content/SearchData?page=#{p}&amp;RCA=1")
    puts agent.page.uri
  end
end