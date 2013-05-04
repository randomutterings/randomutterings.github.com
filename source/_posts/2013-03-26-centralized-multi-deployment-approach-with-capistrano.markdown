---
layout: post
title: "Centralized multi-deployment approach with Capistrano"
date: 2013-03-26 19:04
comments: true
categories: [Capistrano, Deployments, SOA]
---

Capistrano is most often used for deploying a single Rails application but with a few libraries you can use it to deploy multiple applications or “services” in a service oriented architecture.  In this article I'll walk through how to build this type of deployment application with Capistrano.

<!-- more -->

From the docs:

Capistrano is a utility and framework for executing commands in parallel on multiple remote machines, via SSH. It uses a simple DSL (borrowed in part from Rake) that allows you to define tasks, which may be applied to machines in certain roles. It also supports tunneling connections via some gateway machine to allow operations to be performed behind VPN's and firewalls.

Normally with Capistrano your deployment code lives in the application repository.  With multiple applications or services, there would be serious disadvantages to this approach.  With a centralized deployment application, all of our deployment code is located in single repository.  We can easily refactor it, secure it, and it's DRY.

Since Capistrano executes remote commands via SSH, user restrictions inside the deployment application shouldn’t be necessary.  Individuals who have the permissions needed for a particular environment (production, qa, staging, etc) will be able to deploy to that environment.  Additionally, you can restrict access to the deployment application repository so only specific people/departments have access to update the deployment application code.

Here's an example of a directory structure for deploying 3 services in an SOA application.

    ├── config
    │   ├── deploy
    │   │   ├── soa_app
    │   │   │   ├── core_service
    │   │   │   │   ├── production.rb
    │   │   │   │   └── qa.rb
    │   │   │   │   └── staging.rb
    │   │   │   ├── user_service
    │   │   │   │   ├── production.rb
    │   │   │   │   └── qa.rb
    │   │   │   │   └── staging.rb
    │   │   │   └── billing_service
    │   │   │       ├── production.rb
    │   │   │       └── qa.rb
    │   │   │       └── staging.rb
    │   │   └── global.rb
    │   └── deploy.rb
    ├── recipes
    ├── Capfile
    └── Gemfile

You can create this structure manually or use a gem called CapHub which will generate the basic directory structure for you and install some needed gems.  It will also install a couple that are out of scope for this article so we'll build the structure manually.

Add the following to your Gemfile

{% codeblock lang:ruby %}
source "https://rubygems.org"

# Required: allows us to specify a separate configuration for each application/service
gem "capistrano-multiconfig"

# Optional: confirm deployment tasks if the local deployment application is not up to date
gem "capistrano-uptodate"     

# Optional: use rvm to install and manage RVM and Rubies remotely
gem "rvm-capistrano"
{% endcodeblock %}

Add the following to your Capfile

{% codeblock lang:ruby %}
require 'capistrano/multiconfig'
require 'capistrano/uptodate'
require 'rvm/capistrano'

# Optional: Use bundler for managing remote gems
require 'bundler/capistrano'

# Load capistrano's deploy recipes
load 'deploy'
load 'deploy/assets'

# Load all custom recipes
Dir['recipes/**/*.rb'].each { |recipe| load(recipe) }

# Load common configuration
load 'config/global'
{% endcodeblock %}

The core functionality of our deployment application is in capistrano-multiconfig. It automatically builds Capistrano configurations from files located in config/deploy so creating config/deploy/soa_app/user_service/production.rb will make the corresponding configuration task available:

    $ cap -T
    ...
    cap soa_app:user_service:production     # Load soa_app:user_service:production configuration
    ...

Let's take a look at our common configuration.

{% codeblock lang:ruby %}
# config/global.rb
# Put configuration here that is shared between ALL applications to be deployed

# some global defaults, yours may differ
set :rvm_ruby_string, '1.9.3'
set :scm, :git
set :scm_verbose, true
set :user, 'bob'
ssh_options[:keys] = [File.join(ENV["HOME"], ".ssh", "id_rsa")]
set :git_enable_submodules, 1

# keeps a cached copy of the application repo on each environment for faster deployments
set :deploy_via, :remote_cache

# set the application/service name and rails environment from the task naming convention
set(:application) { config_name.split(':').reverse[1] }    # core_service, user_service, billing_service
set(:stage) { config_name.split(':').last }                # production, qa, staging
set(:rails_env) { stage }
  
# we're deploying our database and server config also
before 'deploy:finalize_update', 'deploy:symlink_database_config', 'deploy:symlink_apache_config'

after(:deploy) do
  deploy.migrate        # migrate the database
  deploy.cleanup        # keep only the last 5 releases
end
{% endcodeblock %}

Notice we're deploying server configuration as well as application code.  This is optional of course but I like the approach.  We're managing server configuration for each service in a git submodule called &#8249;service-name&#8250;-config.  As configuration changes are introduced we track them in version control and we can test them against our staging environment.  Each release of the service knows the commit reference it should pull from its config repo.  Server configuration management (with rollback) is now part of our deployment strategy.  We get access management for sensitive server configs built in and we can perform automated testing for configuration changes.

Anything that doesn't apply globally can be specified in individual environment configurations.  Here's an example production configuration for the user_service.

{% codeblock lang:ruby %}
# config/deploy/soa_app/user_service/production.rb
set :domain, 'users.soa_app.com'
role :web, domain
role :app, domain
role :db,  domain, :primary => true
set :repository,  "git@github.com:bobs-code/user_service.git"
  
# ask for a branch or tag to deploy for the user_service
# make sure to, git push origin --tags, first if you wish to deploy a tag
set :branch do
  branch = ENV['BRANCH'] || ENV['TAG']
&#xa0;&#xa0;if branch.nil?
    Capistrano::CLI.ui.ask "Branch or Tag to deploy for the user service: "
  else
    branch
  end
end
set :deploy_to,   "/home/bob/#{application}-#{stage}"
{% endcodeblock %}

You can override Capistrano’s default deploy tasks or write your own from scratch.  In our example application we're going to mostly depend on Capistrano's deploy task.  We'll override the restart task for our environment and add tasks to symlink our database and apache configurations.

{% codeblock lang:ruby %}
  desc "Symlink apache config"
  task :symlink_apache_config do
    run "sudo ln -nfs #{release_path}/config/submodule/apache/#{application}-#{stage} /etc/apache2/sites-enabled/#{application}-#{stage}"
  end

  desc "Restart apache"
  task :restart do
    run "sudo /etc/init.d/apache2 restart"
  end
end
{% endcodeblock %}

To deploy our user_service we can execute

    $ cap soa_app:user_service:production deploy

First time deployments to new servers will require some remote directory setup

    $ cap soa_app:user_service:production deploy:setup

Deploying individual services one by one would be time consuming and error prone so we’ll need a way to deploy multiple services simultaneously.  There is a caveat here, since configurations are actually tasks to Capistrano, we must be careful not to run them in parallel or they'll be merged.  To deploy our entire application you could script the deployment tasks in sequence but let's create an additional recipe to handle this instead.

{% codeblock lang:ruby %}
namespace :deploy_all do
 ...
 task :staging do
   Capistrano::CLI.ui.say "deploying core_service:staging"
   system("cap soa_app:core_service:staging deploy")
   
   Capistrano::CLI.ui.say "deploying user_service:staging"
   system("cap soa_app:user_service:staging deploy")
   
   Capistrano::CLI.ui.say "deploying billing_service:staging"
   system("cap soa_app:billing_service:staging deploy")
 end
 ...
end
{% endcodeblock %}

We’ll need a (dummy) configuration environment for this so create an empty file config/deploy/soa_app.rb.

Now we can deploy to staging by executing

    $ cap soa_app deploy_all:staging

Once you get your deployment application functional, I recommend reading http://jondavidjohn.com/blog/2012/04/cleaning-up-capistrano-deployment-output.

References  
capistrano  
capistrano-multiconfig  
capistrano-uptodate  
rvm-capistrano  
caphub