---
layout: post
title: "Subversion script for Rails developers"
date: 2007-09-19 18:32
comments: false
categories: [Ruby, Rails, Subversion]
---

I read somewhere that good developers use version control.  There's a good article on [HowtoUseRailsWithSubversion](http://wiki.rubyonrails.org/rails/pages/HowtoUseRailsWithSubversion).

So why should I repeat those steps for every new project I work on.

<!-- more -->

There is another [article](http://www.railsonwave.com/railsonwave/2006/12/19/smart-subversion-script-for-rails-projects) which attempted to tackle this problem.  However, you still have to do the initial steps and then copy this script into your project directory and execute it.

The following script takes it a bit farther.  You call it like this.

    rails-svn app_dir repository username

So if I wanted to create a new rails project called toejam, I would execute:

    rails-svn toejam svn://randomutterings.com/projects/toejam/trunk chris

Here's what happens under the hood.

1. After checking to make sure the directory doesn't already exist, your rails app is created with the standard rails command.  (If the directory already exists, you will be asked if you want to continue.  Choosing yes will allow the script to continue but if this script has been run on the same app twice, all of the svn commits will be duplicated.)

2. Your new app is imported to the subversion repository specified.  Authentication is done with the username provided.  You may be prompted for a password from your repository.

3. The original app directory is deleted and a working copy is checked out from the repository into that directory.

4. All the default rails log files are removed and subversion is instructed to ignore those files.  (Log files can get large and we definitely don't need them in version control.)

5. The database.yml is moved to database.example to serve as a template for anyone who checks out the code and any newly created database.yml files are ignored by subversion.  (Doing this will help if you have multiple developers working on the same project in different environments.  It also prevents you from overwriting the database.yml file on the production server, oops!

6. Subversion is told to ignore everything in the tmp folder, the .htaccess files, and the dispatch.fcgi file.

Without further ado, here's the script.  Just copy it and save it somewhere in your path.  /usr/bin is a good choice.  Don't forget to chmod 755 so you can execute it.

{% codeblock lang:bash %}
  #!/bin/bash
  
  if [ "$#" != "3" ]; then
    echo "Usage: rails-svn app_dir repository username"
    exit 1
  fi
  
  APPDIR=./$1
  SVN_TRUNK=$2
  SVN_USER=$3
  
  function check_if_exist () {
    if [[ -e $1 ]]; then
      echo ""
      echo "$1 already exists, overwrite? y or n"
      echo ""
      read OVERWRITE
      case "$OVERWRITE" in
    y)
      echo "Overwriting..."
        ;;
    *)
      echo "Action canceled"
      exit 1
        ;;
      esac
    fi
  }
  
  check_if_exist ${APPDIR}
  rails $APPDIR
  svn import $APPDIR $SVN_TRUNK -m "Import" --username $SVN_USER
  sudo rm -r $APPDIR
  svn checkout $SVN_TRUNK $APPDIR
  cd $APPDIR
  
  svn remove log/*
  svn commit -m "removing log files" 
  svn propset svn:ignore "*.log" log/
  svn update log/
  svn commit -m "Ignoring all files in /log/ ending in .log"
  svn move config/database.yml config/database.example
  svn commit -m "Moving database.yml to database.example to provide a template for anyone who checks out the code"
  svn propset svn:ignore "database.yml" config/
  svn update config/
  svn commit -m "Ignoring database.yml"
  svn remove tmp/*
  svn propset svn:ignore "*" tmp/
  svn update tmp/
  svn commit -m "ignore tmp/ content from now" 
  svn propset svn:ignore ".htaccess" public/
  svn update public/
  svn commit -m "Ignoring .htaccess"
  svn propset svn:ignore "dispatch.fcgi" public/
  svn update public/
  svn commit -m "Ignoring dispatch.fcgi"
{% endcodeblock %}
I'm interested in converting this into a gem and possibly rewriting it in ruby.  If anyone has the skills and wants to contribute, leave a comment or [email me](mailto:randomutterings@gmail.com).