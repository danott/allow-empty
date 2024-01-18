require "bundler/setup"
Bundler.require(:default)
require "rake/clean"

BUILD_DIR = "_site"
CLOBBER.include BUILD_DIR

Commit =
  Struct.new(:hash, :author, :date, :title, :body) do
    def task_name
      "#{BUILD_DIR}/#{hash}/index.html"
    end

    def relative_url
      "/#{hash}/"
    end

    def markdown_for_html_member
      "# #{title}\n\n#{body}"
    end

    def markdown_for_html_collection
      "- [#{title}](#{relative_url})"
    end

    def markdown_for_feed_member
      body
    end
  end

def load_commits
  `git log --pretty`.split(/^commit /)
    .drop(1)
    .map { |string| parse_commit(string) }
end

def parse_commit(string)
  lines = string.lines.map(&:strip)
  hash = lines[0]
  author = lines[1].sub(/Author:\s*/, "")
  date = Time.parse(lines[2].sub(/Date:\s*/, ""))
  title = lines[4].strip
  body = lines.drop(6).map(&:strip).join("\n")

  Commit.new(hash, author, date, title, body)
end

def render_markdown(markdown, options = {})
  Kramdown::Document.new(
    markdown,
    {
      auto_ids: false,
      hard_wrap: false,
      input: "GFM",
      syntax_highlighter: "rouge",
      template: "document"
    }.merge(options)
  ).to_html
end

COMMITS = load_commits.freeze

task build: ["#{BUILD_DIR}/index.html", "#{BUILD_DIR}/feed.xml"]

FEED_COMMITS = COMMITS.first(50)
task "#{BUILD_DIR}/feed.xml" => FEED_COMMITS.map(&:task_name) do |task|
  FileUtils.mkdir_p(BUILD_DIR)
  File.write(task.name, Feed.new(FEED_COMMITS).render)
end

task "#{BUILD_DIR}/index.html" => COMMITS.map(&:task_name) do |task|
  FileUtils.mkdir_p(BUILD_DIR)
  File.write(task.name, render_markdown(INDEX_MARKDOWN))
end

rule ".html" do |task|
  commit = COMMITS.find { |candidate| candidate.task_name == task.name }
  FileUtils.mkdir_p(File.dirname(task.name))
  File.write(task.name, render_markdown(commit.markdown_for_html_member))
end

desc "Run a web server hosting #{BUILD_DIR}"
task :serve do
  server = WEBrick::HTTPServer.new(Port: 3000, DocumentRoot: BUILD_DIR)
  trap("INT") { server.stop }
  server.start
end

Feed =
  Struct.new(:commits) do
    def channel_link
      "https://allow-empty.danott.website"
    end

    def channel_title
      "allow-empty"
    end

    def render
      RSS::Maker.make("2.0") do |maker|
        maker.channel.author = commits.first.author
        maker.channel.updated = commits.map(&:date).max.to_s
        maker.channel.about = channel_link
        maker.channel.link = channel_link
        maker.channel.title = channel_title
        maker.channel.description = channel_title

        commits.each do |commit|
          maker.items.new_item do |item|
            item.description = render_markdown(commit.body, { template: nil })
            item.link = channel_link + commit.relative_url
            item.title = commit.title
            item.updated = commit.date.to_s
          end
        end
      end
    end
  end

INDEX_MARKDOWN = <<~HTML
  # allow-empty

  `allow-empty` is an experimental blog by [@danott](https://github.com/danott/).
  Git commits to the [github.com/danott/allow-empty](https://github.com/danott/allow-empty) repository are the entries.
  
  ```bash
  # Try it for yourself
  git clone git@github.com:danott/allow-empty.git
  cd allow-empty
  git commit --allow-empty
  rake clobber build serve
  ````

  #{COMMITS.map(&:markdown_for_html_collection).join("\n")}
HTML
