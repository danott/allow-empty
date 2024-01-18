require "bundler/setup"
Bundler.require(:default)
require "rake/clean"

BUILD_DIR = "_site"
CLOBBER.include BUILD_DIR

Commit =
  Struct.new(:hash, :author, :date, :title, :body) do
    def self.all
      @all ||=
        `git log --pretty`.split(/^commit /)
          .drop(1)
          .map do |string|
            lines = string.lines.map(&:strip)
            hash = lines[0]
            author = lines[1].sub(/Author:\s*/, "")
            date = lines[2].sub(/Date:\s*/, "")
            title = lines[4].strip
            body = lines.drop(6).map(&:strip).join("\n")

            Commit.new(hash, author, date, title, body)
          end
    end

    def self.for_feed
      all.first(50)
    end

    def task_name
      "#{BUILD_DIR}/#{hash}/index.html"
    end

    def relative_url
      "/#{hash}/"
    end

    def github_url
      "https://github.com/danott/allow-empty/commit/#{hash}/#comments"
    end

    def render(rendering_collection:)
      ERB.new(File.read("commit.erb")).result(self.binding)
    end

    def render_body
      Kramdown::Document.new(
        body,
        {
          auto_ids: false,
          hard_wrap: false,
          input: "GFM",
          syntax_highlighter: "rouge",
          typographic_symbols: {
            hellip: "..."
          }
        }
      ).to_html
    end

    def time
      Time.parse(date)
    end
  end

# Build the entire website
task build: [
       "#{BUILD_DIR}/index.html",
       "#{BUILD_DIR}/feed.xml",
       "#{BUILD_DIR}/style.css"
     ]

# Build feed.xml
task "#{BUILD_DIR}/feed.xml" => Commit.for_feed.map(&:task_name) do |task|
  FileUtils.mkdir_p(BUILD_DIR)
  File.write(task.name, Feed.new(Commit.for_feed).render)
end

# Build index.html
task "#{BUILD_DIR}/index.html" => Commit.all.map(&:task_name) do |task|
  title = "allow-empty"
  html = Layout.new(title, Collection.new(Commit.all).render).render

  FileUtils.mkdir_p(BUILD_DIR)
  File.write(task.name, html)
end

# Copy style.css
task "#{BUILD_DIR}/style.css" => "style.css" do |task|
  FileUtils.mkdir_p(BUILD_DIR)
  FileUtils.cp("style.css", task.name)
end

# Rake rule to generate a page for each commit
rule ".html" do |task|
  commit = Commit.all.find { |candidate| candidate.task_name == task.name }
  title = commit.title
  html = Layout.new(title, commit.render(rendering_collection: false)).render

  FileUtils.mkdir_p(File.dirname(task.name))
  File.write(task.name, html)
end

# Run a web server to preview the site in development
desc "Run a web server hosting #{BUILD_DIR}"
task :serve do
  server = WEBrick::HTTPServer.new(Port: 3000, DocumentRoot: BUILD_DIR)
  trap("INT") { server.stop }
  server.start
end

Layout =
  Struct.new(:title, :body) do
    def render
      ERB.new(File.read("layout.erb")).result(binding)
    end
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
        maker.channel.updated = commits.map(&:time).max.to_s
        maker.channel.about = channel_link
        maker.channel.link = channel_link
        maker.channel.title = channel_title
        maker.channel.description = channel_title

        commits.each do |commit|
          maker.items.new_item do |item|
            item.description = commit.render_body + footer(commit)
            item.link = channel_link + commit.relative_url
            item.title = commit.title
            item.updated = commit.time.to_s
          end
        end
      end
    end

    def footer(commit)
      address = %w[danott hey.com].join("@")
      website_url = channel_link + commit.relative_url
      website_url_without_protocol = website_url.delete_prefix("https://")

      <<~HTML
        <hr />
        <p>
          Thanks for reading via RSS! 
          Wanna talk about it? 
          You can <a href="#{commit.github_url}">comment on GitHub</a> or <a href="mailto:#{address}?subject=Re: #{website_url_without_protocol}">reply via email</a>.
        </p>
      HTML
    end
  end

Collection =
  Struct.new(:commits) do
    def render
      ERB.new(File.read("collection.erb")).result(binding)
    end
  end
