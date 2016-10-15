require 'bundler/gem_tasks'
require 'rspec/core/rake_task'

RSpec::Core::RakeTask.new(:spec)

task default: :spec

task :generate_nextstep_mappings do
  require "net/http"
  url = "http://ftp.unicode.org/Public/MAPPINGS/VENDORS/NEXT/NEXTSTEP.TXT"
  mappings = Net::HTTP.get(URI(url))
    .lines
    .grep(/^[^#$]/)
    .map {|l| l.split("\t", 3) }
    .reduce("module AsciiPlist\n  module Unicode\n    # Taken from #{url}\n    NEXT_STEP_MAPPING = {\n") do |f, (ns, uc, cm)|
      f << "      #{ns} => #{uc}, #{cm}"
    end << "    }.freeze\n  end\nend\n"
  File.open("lib/ascii_plist/unicode/next_step_mapping.rb", "w") {|f| f << mappings}
end

task :generate_quote_maps do
  quote_map = {
    "\a" => "\\a",
    "\b" => "\\b",
    "\f" => "\\f",
    "\r" => "\\r",
    "\t" => "\\t",
    "\v" => "\\v",
    "\n" => "\\n",
    %(') => "\\'",
    %(") => '\\"',
  }

  unquote_map = quote_map.reduce({"\n" => "\n"}) do |unquote_map, (value, escaped)|
    unquote_map[escaped[1..-1]] = value
    unquote_map
  end

  0.upto(31) {|i| quote_map[[i].pack('U')] ||= format("\\U%04x", i) }
  quote_regexp = Regexp.union(quote_map.keys)

  dump_hash = proc do |hash, indent = 4|
    hash.reduce("{\n") {|dumped, (k, v)| dumped << "#{' ' * (indent + 2)}#{k.dump} => #{v.dump},\n" } << ' ' * indent << "}.freeze"
  end

  map = <<-RUBY
module AsciiPlist
  module Unicode
    QUOTE_MAP = #{dump_hash[quote_map]}

    UNQUOTE_MAP = #{dump_hash[unquote_map]}

    QUOTE_REGEXP = #{quote_regexp.inspect}
  end
end
  RUBY

  File.open("lib/ascii_plist/unicode/quote_maps.rb", "w") {|f| f << map }
end
