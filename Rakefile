require 'active_record_migrations'
require 'mechanize'

ActiveRecordMigrations.load_tasks

desc 'Parse'
task :parse => :environment do
  agent = Mechanize.new

  agent.get('http://webprod1.isbe.net/ILEARN/')
  agent.page.forms_with(action: '/ILEARN/Content/SearchData')[0].submit

  (2..87).each do |p|
    puts agent.page.uri
    # --------------------------------------------------------
    agent.page.links_with(href: /(RCDTSeclected)/).each do |l|
      tables = l.click

      puts "\t#{tables.uri}"

      info = tables.search('#DistrictInfo')
      district_number = info.text[/(\d+)/].to_i
      district_name = info.text[/-.*$/].delete_prefix('- ')

      info = tables.search('.col-md-offset-4 p')
      superintendent = info[0].text[/:.*$/].delete_prefix(': ')
      address = info[1].text[/:.*$/].delete_prefix(': ')
      phone = info[2].text[/:.*$/].delete_prefix(': ')
      district_type = info[3].text[/:.*$/].delete_prefix(': ')
      data_source_url = tables.uri
      scrape_dev = 'evilx'

      sql = "INSERT INTO ilearn_districts (district_number, district_name,
            superintendent, address, phone, district_type, data_source_url, scrape_dev)
            VALUES (#{district_number}, '#{district_name}', '#{superintendent}', '#{address}',
            '#{phone}', '#{district_type}', '#{data_source_url}', '#{scrape_dev}');"
      exec_sql(sql)

      info = tables.search('td')
      state_amount = init(info[1])
      local_amount = init(info[5])
      federal_amount = init(info[9])
      total_amount_receint = init(info[13])

      sql = "SELECT id FROM ilearn_districts WHERE district_number = #{district_number}"
      ilearn_district_id = exec_sql(sql)[0]['id']

      sql = "INSERT INTO ilearn_receits_revenues (district_number, district_name,
             state_amount, local_amount, federal_amount, total_amount, data_source_url,
             scrape_dev, ilearn_districts_id) VALUES (#{district_number}, '#{district_name}',
             #{state_amount}, #{local_amount}, #{federal_amount}, #{total_amount_receint}, '#{data_source_url}',
             '#{scrape_dev}', #{ilearn_district_id});"
      exec_sql(sql)

      instruction_amount = init(info[17])
      general_administration_amount = init(info[21])
      support_services_amount = init(info[25])
      other_amount = init(info[29])
      total_amount_exp = init(info[33])

      sql = "INSERT INTO ilearn_expenditures_disbursements (district_number,
            district_name, instruction_amount, general_administration_amount,
            suppert_services_amount, other_amount, total_amount, data_source_url,
            scrape_dev, ilearn_districts_id) VALUES (#{district_number}, '#{district_name}',
            #{instruction_amount}, #{general_administration_amount}, #{support_services_amount},
            #{other_amount}, #{total_amount_exp}, '#{data_source_url}', '#{scrape_dev}', #{ilearn_district_id});"
      exec_sql(sql)

      eight_month = init(info[39])
      statewide_ada = init(info[41]) # may by float
      net_operating = init(info[43])
      operating_expance = init(info[45])
      statewide_oepp_rank = init(info[47])
      statewide_oepp = init(info[49])
      allowance = init(info[51])
      per_capita = init(info[53])
      statewide_pctc_rank = init(info[55])
      statewide_pctc = init(info[57])

      sql = "INSERT INTO ilearn_per_student_info (district_number, district_name,
            \"9-Month Avg. Daily Attendance\", \"Statewide ADA\", \"Net Operating Expanse\",
            \"Operating Expance Per Pupil (OEPP)\", \"Statewide OEPP Rank\", \"Statewide OEPP\",
            \"Allowance for Tuition Computation\", \"Per Capita Tuition Charge (PCTC)\",
            \"Statewide PCTC Rank\", \"Statewide PCTC\", data_source_url, scrape_dev, ilearn_districts_id)
            VALUES (#{district_number}, '#{district_number}', #{eight_month}, #{statewide_ada},
            #{net_operating}, #{operating_expance}, #{statewide_oepp_rank}, #{statewide_oepp}, #{allowance},
            #{per_capita}, #{statewide_pctc_rank}, #{statewide_pctc}, '#{data_source_url}', '#{scrape_dev}',
            #{ilearn_district_id});"
      exec_sql(sql)

      real_eav = init(info[59])
      real_per_pupil = init(info[61])
      statewide_eavpp_rank = init(info[63])
      formula_type = info[65].text
      total_tax_rate = init(info[67])
      statewide_ttr_rank = init(info[69])
      operating_tax_rate = init(info[71]) # may by float
      statewide_otr_rank = init(info[73])

      sql = "INSERT INTO ilearn_tax_information (district_number, district_name,
            real_eav, real_per_pupil, statewide_eavpp_rank, formula_type, total_tax_rate,
            statewide_ttr_rank, operating_tax_rate, statewide_otr_rank)
            VALUES (#{district_number}, '#{district_name}', #{real_eav}, #{real_per_pupil},
            #{statewide_eavpp_rank}, '#{formula_type}', #{total_tax_rate}, #{statewide_ttr_rank},
            #{operating_tax_rate}, #{statewide_otr_rank});"
      exec_sql(sql)

      puts "\tDONE"
    end
    # ------------------------------------------------------------

    agent.get("http://webprod1.isbe.net/ILEARN/Content/SearchData?page=#{p}&amp;RCA=1")
  end
end

def exec_sql(sql)
  ActiveRecord::Base.connection.execute(sql)
end

def init(value)
  return 0 if value.text.eql?('')

  value.text[/(\d.*)|./].delete(',').to_i
end