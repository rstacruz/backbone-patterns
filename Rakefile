repo = ENV['GITHUB_REPO'] || 'rstacruz/backbone-patterns'

desc "Builds documentation"
task :build do
  # github.com/rstacruz/reacco
  analytics = "--analytics #{ENV['ANALYTICS_ID']}"  if ENV['ANALYTICS_ID']
  system "reacco --literate --toc --api lib --github #{repo} #{analytics} -o output --css style.css"
end

desc "Uploads documentation"
task :deploy => :build do
  # github.com/rstacruz/git-update-ghpages
  system "git update-ghpages -i doc #{repo}"
end

task :read => :build do
  system "open output/index.html"
end
