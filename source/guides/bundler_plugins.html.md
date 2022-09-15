---
title: How to write a Bundler plugin
---

.container.guide
  %h1#how-to-write-a-bundler-plugin
    How to write a Bundler plugin

  %h2#what-is-a-plugin
    What is a plugin?

  .contents
    .bullet
      .description
        Bundler plugins are specialized gems that are aimed at integrating and extending Bundler's functionality.

        This guide will help you start writing your own Bundler plugins.

  %h2#why-would-i-use-a-plugin
    Why would I use a plugin?

  .contents
    .bullet
      .description
      Plugins are able to integrate with and extend Bundler.

      %p
        Currently, a plugin is able to:

      %ul
        %li
          Add commands to Bundler (e.g. <code>bundle my_command</code>)
        %li
          Add a special handler to install a gem (e.g. Mercurial or SVN)
        %li
          Add functionality to specific hook points:

          %ul

            %li
              A list of all available hooks, their descriptions, and their block arguments are available

              = link_to 'in the plugin/events.rb file.', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/plugin/events.rb'

            %li
              Note: Make sure to check out the <code>events.rb</code> file in the version of Bundler you are using.


  %h2#using-a-plugin
    Using a plugin

  .contents
    .bullet
      .description

      %h4#install-from-command
        Install a plugin from a command

      %p
        Plugins can be installed from RubyGems (default) or from a Git server.
        To use a gem on your machine, you can run <code>bundler plugin install gem_name</code>.
        Once the plugin is installed, the commands will be available for use and the hooks will be automatically registered with Bundler.

      %p
        Run <code>bundler plugin help install</code> for more details help and instructions on installing from Git.

      %p
        In Bundler 2.2.0, you can uninstall with <code>bundler plugin uninstall gem_name</code>.

      %h4#install-from-gemfile
        Install a plugin from your Gemfile

      %p
        You can also specify a plugin in your Gemfile:

      :code
        # lang: ruby
        plugin 'my_plugin' # Installs from Rubygems
        plugin 'my_plugin', path: '/path/to/plugin' # Installs from a path
        plugin 'my_plugin', git: 'https://github.com:repo/my_plugin.git' # Installs from Git
  %h2#getting-started-with-development
    Getting started with development

  .contents
    .bullet
      .description

      %h3#create-a-gem
        1. Create a gem

      %p
        You'll first need to create a specialized gem before you can make a Bundler plugin.

      %p
        = link_to 'Create a gem using this guide.', './creating_gem.html'
        When you're done, come back to this guide and move onto step two.

      %h3#plugins-rb{ id: 'plugins_rb' }
        2. Create a plugins.rb file

      %p
        A <code>plugins.rb</code> file is located at the top level of your gem's folder and is the entry point Bundler will use to call your plugin.
        This is a Ruby file that defines your commands, hooks, and other code. Often, you may just require the gem's upper-most lib file.

      %p
        For example, if your gem is called "my_plugin", you might have a file at <code>lib/my_plugin.rb</code> which contains the highest level namespace for your gem.
        Your <code>plugins.rb</code> file might be:

        :code
          # lang: ruby
          require 'my_plugin'
      %p
        The <code>lib/my_plugin.rb</code> file would include other require statements, hooks, and commands similar to a normal gem.
      %h3#developing-your-plugin-commands{ id: 'developing_your_plugin_commands' }
        3. Making Bundler commands

      %p
        Bundler commands allow you to extend the Bundler interface with additional functionality.

      %p
        To add a Bundler command, you need to make a class that registers itself (or another class) as a command.
        For example, to add support for a <code>bundler my_command</code> command, you might create a class like so:

      :code
        # lang: ruby
        class MyCommand < Bundler::Plugin::API
          # Register this class as a handler for the `my_command` command
          command "my_command"

          # The exec method will be called with the `command` and the `args`.
          # This is where you should handle all logic and functionality
          def exec(command, args)
            if args.empty?
              # Using BundlerError in plugins is recommended. See below.
              raise BundlerError, 'My plugin requires arguments'
            end
            puts "You called " + command + " with args: " + args.inspect
          end
        end
      or

      :code
        # lang: ruby
        module MyCommand
          # Register this class as a handler for the `my_command` command
          Bundler::Plugin::API.command('my_command', self)

          # The exec method will be called with the `command_name` and the `args`.
          # This is where you should handle all logic and functionality
          def exec(command_name, args)
            puts "You called " + command_name + " with args: " + args.inspect
          end
        end
      %p
        These two elements are important in order for a command to register in Bundler:

      %ol
        %li
          <code>Bundler::Plugin::API.command(COMMAND_NAME, CLASS)</code> or <code>command 'COMMAND_NAME'</code> is called, depending on the method used (see examples above)
        %li
          The class defines the instance method <code>exec(command_name, args)</code>

      %h4#raising-errors{ id: 'raising_errors' }
        Raising Errors
      %p
        If something goes wrong, your plugins should raise a `BundlerError`.
        It's not recommended to raise e.g. `Exception` in a plugin, because that will cause Bundler to print its own bug report template, asking users to report the bug to Bundler itself.
      %p
        To see in detail how bundler rescues errors, check out `bundler/friendly_errors.rb`.

      %h3#developing-your-plugin-hooks{ id: 'developing_your_plugin_hooks' }
        4. Using Bundler hooks

      %p
        To interface with various parts of Bundler, you can use a hook.
        Hooks will let you inject some functionality at specific events by registering to listen for specific things to happen.
        To listen to an event, you need to add a hook for it and provide a block.

      %p
        For example, for a <code>Bundler::Plugin::Events::GEM_BEFORE_INSTALL_ALL</code> hook you must give a block that has an argument for an Array of <code>Bundler::Dependency</code> objects:

      :code
        # lang: ruby
        Bundler::Plugin.add_hook('before-install-all') do |dependencies|
          # Do something with the dependencies
        end
      %h3#developing-your-plugin-sources{ id: 'developing_your_plugin_sources' }
        5. Developing a source plugin

      %p
        A source plugin allows you to specify more possible installation sources to use within Bundler.
        For example, let's say you want to install gems from Amazon S3. This can be done by building a plugin.

      %p
        It is recommended to get familiar with the API for <code>Bundler::Plugin::API::Source</code> which is available

        = link_to 'on rubydoc.info', 'https://www.rubydoc.info/github/bundler/bundler/Bundler/Plugin/API/Source'

        or

        = link_to 'in the source code.', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/plugin/api/source.rb'

      %p
        The basic overview of the source plugin is that you must subclass <code>Bundler::Plugin::API::Source</code> and override a number of methods.
        Those methods are indicated in the docs/source code linked above.

      %p
        Bundler uses the source plugin API to provide interfaces for RubyGems, Git, and path-based gems. The source code for these pieces may prove useful in understanding the API:

      %ul
        %li
          = link_to 'RubyGems source', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/rubygems.rb'
        %li
          = link_to 'git source', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/git.rb'
        %li
          = link_to 'path source', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/path.rb'

      %h3#running-your-plugin-locally{ id: 'running_your_plugin_locally' }
        6. Running your plugin locally

      %p
        To install and run your plugin locally, you can run <code>bundler plugin install --git '/PATH/TO/GEM' copycat</code>

      %h3#deploying-your-plugin{ id: 'deploying_your_plugin' }
        7. Deploying your plugin

      %p
        Deploy your plugin to RubyGems so others can install it. For instructions on deploying to RubyGems, visit

        = link_to 'this guide.', './creating_gem.html#releasing-the-gem'

      %p
        Although plugins can be installed from a git branch, it's recommended to install plugins directly from RubyGems.

    %h2#examples
      Example Plugins

    .contents
      .bullet
        .description
        Here are a few plugins that you can use as examples and inspiration:

        %ul
          %li
            For a plugin that adds a command, take a look at
            = link_to 'rubysec/bundler-audit', 'https://github.com/rubysec/bundler-audit'
          %li
            For a plugin that makes use of hooks, take a look at
            = link_to 'jules2689/extended_bundler-errors', 'https://github.com/jules2689/extended_bundler-errors'
          %li
            For an example of source plugin, take a look at Bundler's implementations for

            = link_to 'the RubyGems source,', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/rubygems.rb'
            = link_to 'the git source,', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/git.rb'
            and
            = link_to 'the path source', 'https://github.com/rubygems/rubygems/blob/master/bundler/lib/bundler/source/path.rb'
