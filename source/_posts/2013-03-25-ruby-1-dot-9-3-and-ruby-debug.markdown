---
layout: post
title: "ruby-1.9.3 and ruby-debug"
date: 2012-03-24 17:52
comments: false
categories: ruby
---

To get `ruby-debug` working with `ruby-1.9.3-p0` you have to use some as yet
unpublished versions of `ruby-debug-base19` (0.11.26) and it's dependency,
`linecache19` (0.5.13).  Additionally, you'll need to apply the aptly named
"debug" patch to `ruby-1.9.3-p0`.  I don't recall the specific reason the gems
are not yet available on [rubygems.org](http://rubygems.org) (as of
2012-02-22), but until such time, here's how you get `ruby-debug` working with
version `ruby-1.9.3-p0`.

_Note_: If you're not using `rvm`, shame on you. You'll need to apply the patch
manually, which I won't detail.  The patch can be found
[here](https://github.com/ruby/ruby/pull/47).

Apply the "debug" patch:

    $ rvm reinstall 1.9.3-p0 --patch debug --force-autoconf

Point your favorite editor at your `Gemfile` and add:

    gem 'ruby-debug19', :require => false
    gem 'ruby-debug-base19', :git => 'https://github.com/tribune/ruby-debug-base19.git', :require => false
    gem 'linecache19', :git => 'git@github.com:chuckg/linecache19.git', :branch => "0_5_13/dependencies", :require => false
   
And now the secret sauce to allow `ruby-debug-base19` to compile correctly:

    $ bundle config build.ruby-debug-base19 --with-ruby-include=$rvm_path/src/ruby-1.9.3-p0/

Without the above command you'll receive the
`Gem::Installer::ExtensionBuildError` error listed below. If you're working
with a team, it may be useful to check the bundle config into your repository.
Check out `bundle help config` for more details. 

##### rdebug

The caveat to using git as the gem source for our dependencies is that they
are installed in a separate (non-standard) gem path, causing `rdebug` to puke
when it can't find them. `bundler` to the rescue:

    $ bundle exec rdebug ...

This executes `rdebug` within the context of the bundle, thereby including the
necessary dependencies. 

## Common errors:

For reference (and the glory of search) here are some common errors you might have seen during your journey here. 

##### ruby-debug

```bash
$ bundle install
...
Could not find rbx-require-relative-0.0.6 in any of the sources
```
_You're still using the `ruby-debug` gem, not `ruby-debug19`._

```bash
$ rdebug -Itest test/unit/model_test.rb
/Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/dependency.rb:247:in `to_specs': Could not find linecache19 (>= 0.5.11) amongst [...] (Gem::LoadError)
	from /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/specification.rb:771:in `block in activate_dependencies'
	from /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/specification.rb:760:in `each'
	from /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/specification.rb:760:in `activate_dependencies'
	from /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/specification.rb:744:in `activate'
	from /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems.rb:1209:in `gem'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/bin/rdebug:18:in `<main>'
```
_Trying using: `bundle exec rdebug ...`_

##### ruby-debug-base19

```bash
Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

        /Users/rubyist/.rvm/rubies/ruby-1.9.3-p0/bin/ruby extconf.rb 
checking for rb_method_entry_t.called_id in method.h... no
checking for rb_control_frame_t.method_id in method.h... no
extconf.rb:16:in `block in <main>': break from proc-closure (LocalJumpError)
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby_core_source-0.1.5/lib/ruby_core_source.rb:18:in `call'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby_core_source-0.1.5/lib/ruby_core_source.rb:18:in `create_makefile_with_core'
	from extconf.rb:32:in `<main>'


An error occured while installing ruby-debug-base19 (0.11.26), and Bundler cannot continue.
Make sure that `gem install ruby-debug-base19 -v '0.11.26'` succeeds before bundling.
```
_This is fixed by adding the build flags to your bundle config._

```ruby
1.9.3-p0 :001 > require 'ruby-debug'
LoadError: dlopen(/Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug-base19-0.11.25/lib/ruby_debug.bundle, 9): Symbol not found: _ruby_threadptr_data_type
  Referenced from: /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug-base19-0.11.25/lib/ruby_debug.bundle
  Expected in: flat namespace
 in /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug-base19-0.11.25/lib/ruby_debug.bundle - /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug-base19-0.11.25/lib/ruby_debug.bundle
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `block in require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:236:in `load_dependency'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug-base19-0.11.25/lib/ruby-debug-base.rb:1:in `<top (required)>'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `block in require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:236:in `load_dependency'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/ruby-debug19-0.11.6/cli/ruby-debug.rb:5:in `<top (required)>'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `block in require'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:236:in `load_dependency'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/activesupport-3.2.1/lib/active_support/dependencies.rb:251:in `require'
	from (irb):1
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/railties-3.2.1/lib/rails/commands/console.rb:47:in `start'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/railties-3.2.1/lib/rails/commands/console.rb:8:in `start'
	from /Users/rubyist/.rvm/gems/ruby-1.9.3-p0@railsapp/gems/railties-3.2.1/lib/rails/commands.rb:41:in `<top (required)>'
	from script/rails:6:in `require'
```
_Seeing this? You're probably still using `ruby-debug-base19` (0.11.25) and `linecache19` (0.5.12).  Make sure you manually added the correct versions to your `Gemfile`._
