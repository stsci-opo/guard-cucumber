# Guard::Cucumber [![Build Status](https://secure.travis-ci.org/netzpirat/guard-cucumber.png)](http://travis-ci.org/netzpirat/guard-cucumber)

Guard::Cucumber allows you to automatically run Cucumber features when files are modified.

Tested on MRI Ruby 1.8.7, 1.9.2, 1.9.3, REE and the latest versions of JRuby & Rubinius.

If you have any questions please join us on our [Google group](http://groups.google.com/group/guard-dev) or on `#guard` (irc.freenode.net).

## Install

The simplest way to install Guard is to use [Bundler](http://gembundler.com/).
Please make sure to have [Guard](https://github.com/guard/guard) installed before continue.

Add Guard::Cucumber to your `Gemfile`:

```bash
group :development do
  gem 'guard-cucumber'
end
```

Add the default Guard::Cucumber template to your `Guardfile` by running:

```bash
$ guard init cucumber
```

## Usage

Please read the [Guard usage documentation](https://github.com/guard/guard#readme).

## Guardfile

Guard::Cucumber can be adapted to all kind of projects and comes with a default template that looks like this:

```ruby
guard 'cucumber' do
  watch(%r{^features/.+\.feature$})
  watch(%r{^features/support/.+$})                      { 'features' }
  watch(%r{^features/step_definitions/(.+)_steps\.rb$}) { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'features' }
end
```

Expressed in plain English, this configuration tells Guard::Cucumber:

1. When a file within the features directory that ends in feature is modified, just run that single feature.
2. When any file within features/support directory is modified, run all features.
3. When a file within the features/step_definitions directory that ends in \_steps.rb is modified,
run the first feature that matches the name (\_steps.rb replaced by .feature) and when no feature is found,
then run all features.

Please read the [Guard documentation](http://github.com/guard/guard#readme) for more information about the Guardfile DSL.

## Options

You can pass any of the standard Cucumber CLI options using the :cli option:

```ruby
guard 'cucumber', :cli => '-c --drb --port 1234 --profile guard'
```

Former `:color`, `:drb`, `:port` and `:profile` options are thus deprecated and have no effect anymore.

### List of available options

```ruby
:cli => '--profile guard -c'      # Pass arbitrary Cucumber CLI arguments,
                                  # default: '--no-profile --color --format progress --strict'

:feature_sets =>                  # Use non-default feature directory/ies
  ['set_a', 'set_b']              # default: ['features']

:bundler => false                 # Don't use "bundle exec" to run the Cucumber command
                                  # default: true

:binstubs => true                 # use "bin/cucumber" to run the Cucumber command (implies :bundler => true)
                                  # default: false

:rvm => ['1.8.7', '1.9.2']        # Directly run your features on multiple ruby versions
                                  # default: nil

:notification => false            # Don't display Growl (or Libnotify) notification
                                  # default: true

:all_after_pass => false          # Don't run all features after changed features pass
                                  # default: true

:all_on_start => false            # Don't run all the features at startup
                                  # default: true

:keep_failed => false             # Keep failed features until they pass
                                  # default: true

:run_all => { :cli => "-p" }      # Override any option when running all specs
                                  # default: {}

:change_format => 'pretty'        # Use a different cucumber format when running individual features
                                  # This replaces the Cucumber --format option within the :cli option
                                  # default: nil

:command_prefix => 'xvfb-run'     # Add a prefix to the cucumber command such as 'xvfb-run' or any
                                  # other shell script.
                                  # The example generates: 'xvfb-run bundle exec cucumber ...'
                                  # default: nil

:focus_on => 'dev'                # Focus on scenarios tagged with '@dev'
                                  # If '@dev' is on line 6 in 'foo.feature',
                                  # this example runs: 'bundle exec cucumber foo.feature:6'
                                  # default: nil

                                  :paths_from_profile => true       # Instead of receiving path information from Guard,
                                                                    # it uses the information from the Cucumber profile to determine
                                                                    # which features/scenarios to run.
                                                                    # default: false
```

## Cucumber configuration

It's **very important** that you understand how Cucumber gets configured, because it's often the origin of
strange behavior of guard-cucumber.

Cucumber uses [cucumber.yml](https://github.com/cucumber/cucumber/wiki/cucumber.yml) for defining profiles
of specific run configurations. When you pass configurations through the `:cli` option but don't include a
specific profile with `--profile`, then the configurations from the `default` profile are also used.

For example, when you're using the default cucumber.yml generated by [cucumber-rails](https://github.com/cucumber/cucumber-rails),
then the default profile forces guard-cucumber to always run all features, because it appends the `features` folder.

### Configure Cucumber solely from Guard

If you want to configure Cucumber from Guard solely, then you should pass `--no-profile` to the `:cli` option.

Since guard-cucumber version 0.3.2, the default `:cli` options are:

```ruby
:cli => '--no-profile --color --format progress --strict'
```

This default configuration has been chosen to avoid strange behavior when mixing configurations form
the cucumber.yml default profile with the guard-cucumber `:cli` option.

You can safely remove `config/cucumber.yml`, since all configuration is done in the `Guardfile`.

### Use a separate Guard profile

If you're using different profiles with Cucumber then you should create a profile for Guard in cucumber.yml,
something like this:

```yaml
guard: --format progress --strict --tags ~@wip
```

Now you want to make guard-cucumber use that profile by passing `--profile guard` to the `:cli`.

## Cucumber with Spork

To use Guard::Cucumber with [Spork](https://github.com/timcharper/spork), you should install
[Guard::Spork](https://github.com/guard/guard-spork) and use the following configuration:

```ruby
guard 'spork' do
  watch('config/application.rb')
  watch('config/environment.rb')
  watch(%r{^config/environments/.*\.rb$})
  watch(%r{^config/initializers/.*\.rb$})
  watch('spec/spec_helper.rb')
end

guard 'cucumber', :cli => '--drb --format progress --no-profile' do
  watch(%r{^features/.+\.feature$})
  watch(%r{^features/support/.+$})                      { 'features' }
  watch(%r{^features/step_definitions/(.+)_steps\.rb$}) { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'features' }
end
```

There is a section with alternative configurations on the [Wiki](https://github.com/netzpirat/guard-cucumber/wiki/Spork-configurations).

## Cucumber with Zeus

To use Guard::Cucumber with [Zeus](https://github.com/burke/zeus), just set the command prefix:
```
guard 'cucumber', :command_prefix => 'zeus', :bundler => false do
  ...
end
```

You need to set `:bundler => false` to avoid using Bundler, as recommended in the Zeus documenation.

## Cucumber with Spring

To use Guard::Cucumber with [Spring](https://github.com/jonleighton/spring), just set the command prefix:
```
guard 'cucumber', :command_prefix => 'spring', :bundler => false do
  ...
end
```

You need to set `:bundler => false` to avoid using Bundler, as recommended in the Spring documenation.

Issues
------

You can report issues and feature requests to [GitHub Issues](https://github.com/netzpirat/guard-cucumber/issues). Try to figure out
where the issue belongs to: Is it an issue with Guard itself or with Guard::Cucumber? Please don't
ask question in the issue tracker, instead join us in our [Google group](http://groups.google.com/group/guard-dev) or on
`#guard` (irc.freenode.net).

When you file an issue, please try to follow to these simple rules if applicable:

* Make sure you run Guard with `bundle exec` first.
* Add debug information to the issue by running Guard with the `--debug` option.
* Add your `Guardfile` and `Gemfile` to the issue.
* Make sure that the issue is reproducible with your description.

## Development

- Documentation hosted at [RubyDoc](http://rubydoc.info/github/guard/guard-cucumber/master/frames).
- Source hosted at [GitHub](https://github.com/netzpirat/guard-cucumber).

Pull requests are very welcome! Please try to follow these simple rules if applicable:

* Please create a topic branch for every separate change you make.
* Make sure your patches are well tested.
* Update the [Yard](http://yardoc.org/) documentation.
* Update the README.
* Update the CHANGELOG for noteworthy changes.
* Please **do not change** the version number.

For questions please join us in our [Google group](http://groups.google.com/group/guard-dev) or on
`#guard` (irc.freenode.net).

## Author

Developed by Michael Kessler, sponsored by [mksoft.ch](https://mksoft.ch).

If you like Guard::Cucumber, you can watch the repository at [GitHub](https://github.com/netzpirat/guard-cucumber)
and follow [@netzpirat](https://twitter.com/#!/netzpirat) on Twitter for project updates.

## Contributors

See the GitHub list of [contributors](https://github.com/netzpirat/guard-cucumber/contributors).

Since guard-cucumber is very close to guard-rspec, some contributions by the following authors have been
incorporated into guard-cucumber:

* [Andre Arko](https://github.com/indirect)
* [Thibaud Guillaume-Gentil](https://github.com/thibaudgg)

## Acknowledgment

The [Guard Team](https://github.com/guard/guard/contributors) for giving us such a nice pice of software
that is so easy to extend, one *has* to make a plugin for it!

All the authors of the numerous [Guards](http://github.com/guard) available for making the Guard ecosystem
so much growing and comprehensive.

## License

(The MIT License)

Copyright (c) 2010-2013 Michael Kessler

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
