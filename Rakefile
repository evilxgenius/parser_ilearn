require 'active_record_migrations'
require 'mechanize'

ActiveRecordMigrations.load_tasks

desc 'Parse'
task :parse => :environment do
  agent = Mechanize.new

  agent.get('http://webprod1.isbe.net/ILEARN/')
  agent.page.forms_with(action: '/ILEARN/Content/SearchData')[0].submit

  (1..86).each do |p|
    agent.get("http://webprod1.isbe.net/ILEARN/Content/SearchData?page=#{p}&amp;RCA=1")
    puts agent.page.uri

    # --------------------------------------------------------
    agent.page.links_with(href: /(RCDTSeclected)/).each do |l|
      tables = l.click

      puts "\t#{tables.uri}"

      info = tables.search('#DistrictInfo')
      district_number = info.text[/(\d+[a-zA-Z]*\d+)/]
      district_name = info.text[/-.*$/].delete_prefix('- ')

      info = tables.search('.col-md-offset-4 p')
      superintendent = init_str(info[0])
      address = init_str(info[1])
      phone = init_str(info[2])
      district_type = init_str(info[3])
      data_source_url = tables.uri
      scrape_dev = 'evilx'

      sql = "INSERT INTO ilearn_districts (district_number, district_name,
            superintendent, address, phone, district_type, data_source_url, scrape_dev)
            VALUES ('#{district_number}', '#{district_name}', '#{superintendent}', '#{address}',
            '#{phone}', '#{district_type}', '#{data_source_url}', '#{scrape_dev}');"
      exec_sql(sql)

      info = tables.search('td')
      state_amount = init_num(info[1])
      local_amount = init_num(info[5])
      federal_amount = init_num(info[9])
      total_amount_receint = init_num(info[13])

      sql = "SELECT id FROM ilearn_districts WHERE district_number = '#{district_number}'"
      ilearn_district_id = exec_sql(sql).first[0]

      sql = "INSERT INTO ilearn_receits_revenues (district_number, district_name,
             state_amount, local_amount, federal_amount, total_amount, data_source_url,
             scrape_dev, ilearn_districts_id)
             VALUES ('#{district_number}', '#{district_name}',
             #{state_amount}, #{local_amount}, #{federal_amount}, #{total_amount_receint}, '#{data_source_url}',
             '#{scrape_dev}', #{ilearn_district_id});"
      exec_sql(sql)

      instruction_amount = init_num(info[17])
      general_administration_amount = init_num(info[21])
      support_services_amount = init_num(info[25])
      other_amount = init_num(info[29])
      total_amount_exp = init_num(info[33])

      sql = "INSERT INTO ilearn_expenditures_disbursements (district_number,
            district_name, instruction_amount, general_administration_amount,
            suppert_services_amount, other_amount, total_amount, data_source_url,
            scrape_dev, ilearn_districts_id)
            VALUES ('#{district_number}', '#{district_name}',
            #{instruction_amount}, #{general_administration_amount}, #{support_services_amount},
            #{other_amount}, #{total_amount_exp}, '#{data_source_url}', '#{scrape_dev}', #{ilearn_district_id});"
      exec_sql(sql)

      eight_month = init_num(info[39])
      statewide_ada = init_num(info[41])
      net_operating = init_num(info[43])
      operating_expance = init_num(info[45])
      statewide_oepp_rank = init_num(info[47])
      statewide_oepp = init_num(info[49])
      allowance = init_num(info[51])
      per_capita = init_num(info[53])
      statewide_pctc_rank = init_num(info[55])
      statewide_pctc = init_num(info[57])

      sql = "INSERT INTO ilearn_per_student_info (district_number, district_name,
            eigth_month_avg_daily_Attendance, statewide_ADA, net_perating_xpanse,
            operating_expance_per_pupil_OEPP, statewide_OEPP_rank, statewide_OEPP,
            allowance_for_tuition_computation, per_capita_tuition_charge_PCTC, Statewide_PCTC_rank,
            statewide_PCTC, data_source_url, scrape_dev, ilearn_districts_id)
            VALUES ('#{district_number}', '#{district_name}', #{eight_month}, #{statewide_ada},
            #{net_operating}, #{operating_expance}, #{statewide_oepp_rank}, #{statewide_oepp}, #{allowance},
            #{per_capita}, #{statewide_pctc_rank}, #{statewide_pctc}, '#{data_source_url}', '#{scrape_dev}',
            #{ilearn_district_id});"
      exec_sql(sql)

      real_eav = init_num(info[59])
      real_per_pupil = init_num(info[61])
      statewide_eavpp_rank = init_num(info[63])
      formula_type = info[65].text
      total_tax_rate = init_num(info[67])
      statewide_ttr_rank = init_num(info[69])
      operating_tax_rate = init_num(info[71])
      statewide_otr_rank = init_num(info[73])

      sql = "INSERT INTO ilearn_tax_information (district_number, district_name,
            real_eav, real_per_pupil, statewide_eavpp_rank, formula_type, total_tax_rate,
            statewide_ttr_rank, operating_tax_rate, statewide_otr_rank)
            VALUES ('#{district_number}', '#{district_name}', #{real_eav}, #{real_per_pupil},
            #{statewide_eavpp_rank}, '#{formula_type}', #{total_tax_rate}, #{statewide_ttr_rank},
            #{operating_tax_rate}, #{statewide_otr_rank});"
      exec_sql(sql)

      puts "\tDONE"
    end
    # ------------------------------------------------------------
  end
end

def exec_sql(sql)
  ActiveRecord::Base.connection.execute(sql)
end

def init_str(value)
  value.text[/:.*$/]&.delete_prefix(': ')
end

def init_num(value)
  value.text[/(\d.*)/]&.delete(',').to_f
end