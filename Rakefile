desc "build and deploy the blog"
task :deploy do
  system "jekyll b"
  system "git add _site -f"
  system "git commit -m 'auto build and deploy commit'"
  system "git subtree push --prefix _site origin master"
end
