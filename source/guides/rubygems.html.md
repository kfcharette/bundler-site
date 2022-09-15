.container.guide
  %h2 Using Bundler while developing a gem

  .contents
    .bullet
      .description
        If you're creating a gem from scratch, you can use bundler's built in gem skeleton to create a base gem for you to edit.
      .how
        :code
          $ bundle gem my_gem
        .notes
          This will create a new directory named <code>my_gem</code> with your new gem skeleton.
    .bullet
      .description
        If you already have a gem, you can create a Gemfile and use Bundler to manage your development dependencies. Here's an example.
      :code
        # lang: ruby
        source "https://rubygems.org"

        gemspec

        gem "rspec", "~> 3.9"
        gem "rubocop", "0.79.0"
      .notes
        In this Gemfile, the `gemspec` method imports gems listed with `add_runtime_dependency` in the `my_gem.gemspec` file, and it also installs rspec and rubocop to test and develop the gem.
        All dependencies from the gemspec and Gemfile will be installed by `bundle install`, but rspec and rubocop will not be included by `gem install mygem` or `bundle add mygem`.
    .bullet
      .description
        Runtime dependencies in your gemspec are treated as if they are listed in your Gemfile, and development dependencies are added by default to the group, <code>:development</code>.
        You can change that group with the <code>:development_group</code> option
      :code
        # lang: ruby
        gemspec :development_group => :dev
    .bullet
      .description
        As well, you can point to a specific gemspec using <code>:path</code>. If your gemspec is in <code>/gemspec/path</code>, use
      :code
        # lang: ruby
        gemspec :path => '/gemspec/path'
    .bullet
      .description
        If you have multiple gemspecs in the same directory, specify which one you'd like to reference using <code>:name</code>
      :code
        # lang: ruby
        gemspec :name => 'my_awesome_gem'
      .notes
        This will use <code>my_awesome_gem.gemspec</code>

    .bullet
      .description
        That's it! Use bundler when developing your gem, and otherwise, use gemspecs normally!
      :code
        $ gem build my_gem.gemspec
