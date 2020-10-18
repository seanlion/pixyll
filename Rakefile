# For Travis CI
require 'rake'
require 'date'
require 'yaml'

CONFIG = YAML.load(File.read('_config.yml'))
USERNAME = CONFIG["username"]
REPO = CONFIG["repo"]
SOURCE_BRANCH = CONFIG["branch"]
DESTINATION_BRANCH = "main"

def check_destination
  unless Dir.exist? CONFIG["destination"]
    sh "git clone https://$GIT_NAME:$GITHUB_TOKEN@github.com/#{USERNAME}/#{REPO}.git #{CONFIG["destination"]}"
  end
end

namespace :site do
  desc "Generate the site"
  task :build do
    check_destination
    sh "bundle exec jekyll build"
  end

  desc "Generate the site and serve locally"
  task :serve do
    check_destination
    sh "bundle exec jekyll serve"
  end

  desc "Generate the site, serve locally and watch for changes"
  task :watch do
    sh "bundle exec jekyll serve --watch"
  end

  desc "Generate the site and push changes to remote origin"
  task :deploy do
    # Detect pull request
    if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
      puts 'Pull request detected. Not proceeding with deploy.'
      exit
    end

    # Configure git if this is run in Travis CI
    if ENV["TRAVIS"]
      sh "git config --global user.name $GIT_NAME"
      sh "git config --global user.email $GIT_EMAIL"
      sh "git config --global push.default simple"
    end

    # Make sure destination folder exists as git repo
    check_destination

    sh "git checkout #{SOURCE_BRANCH}"
    Dir.chdir(CONFIG["destination"]) { sh "git checkout main" }

    # Generate the site
    sh "bundle exec jekyll build"

    # Commit and push to github
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir(CONFIG["destination"]) do
      # check if there is anything to add and commit, and pushes it
      sh "if [ -n '$(git status)' ]; then
            git add --all .;
            git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.';
            git push https://$GITHUB_TOKEN@github.com/#{USERNAME}/#{USERNAME}.github.io.git #{DESTINATION_BRANCH} --quiet ;
         fi"
      puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
    end
  end
end

#############

drafts_dir = '_drafts'
posts_dir  = '_posts'

# rake post['my new post']
desc 'create a new post with "rake post[\'post title\']"'
task :post, :title do |t, args|
  if args.title
    title = args.title
  else
    puts "Please try again. Remember to include the filename."
  end
  mkdir_p "#{posts_dir}"
  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.downcase.gsub(/[^\w]+/, '-')}.md"
  puts "Creating new post: #{filename}"
  File.open(filename, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    date: #{Time.new.strftime('%Y-%m-%d %H:%M')}
    categories:
    ---

    EOS
  end

# Uncomment the line below if you want the post to automatically open in your default text editor
#  system ("#{ENV['EDITOR']} #{filename}")
end

# usage: rake draft['my new draft']
desc 'create a new draft post with "rake draft[\'draft title\']"'
task :draft, :title do |t, args|
  if args.title
    title = args.title
  else
    puts "Please try again. Remember to include the filename."
  end
  mkdir_p "#{drafts_dir}"
  filename = "#{drafts_dir}/#{title.downcase.gsub(/[^\w]+/, '-')}.md"
  puts "Creating new draft: #{filename}"
  File.open(filename, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    date: #{Time.new.strftime('%Y-%m-%d %H:%M')}
    categories:
    ---

    EOS
  end

# Uncomment the line below if you want the draft to automatically open in your default text editor
# system ("#{ENV['EDITOR']} #{filename}")
end

desc 'preview the site with drafts'
task :preview do
  puts "## Generating site"
  puts "## Stop with ^C ( <CTRL>+C )"
  system "jekyll serve --watch --drafts"
end

desc 'list tasks'
task :list do
  puts "Tasks: #{(Rake::Task.tasks - [Rake::Task[:list]]).join(', ')}"
  puts "(type rake -T for more detail)\n\n"
end