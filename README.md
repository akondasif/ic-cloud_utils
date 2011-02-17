# DESCRIPTION

Installs a system-wide RVM and manages installed rubies.

# REQUIREMENTS

## Platform

For the moment this is targeted at Ubuntu, other platforms to be tested.

## Cookbooks

TBD

# ATTRIBUTES:

## `default_ruby`

The default ruby for RVM. If the RVM ruby is not installed, it will be
built as a pre-requisite. The value can also contain a gemset in the form of
`ruby-1.8.7-p330@awesome`.

**Note:** a fully qualified RVM string name needs to be used, which can be
found when running `rvm list known`.

The default is `ruby-1.9.2-p136`.

## `rubies`

A list of RVM rubies to be built and installed. If this list is emptied then
no rubies (not even the default) will be built and installed, like so:

    node[:rvm][:rubies] = []

**Note:** a fully qualified RVM string name needs to be used, which can be
found when running `rvm list known`.

The default is the value of `node[:rvm][:default_ruby]`.

## `global_gems`

A list of gem hashes to be installed into the *global* gemset in each
installed RVM ruby. The RVM global.gems files will be added to and all
installed rubies will be iterated over to ensure full installation coverage.
See the `rvm_gem` resource for more details about the options for each
gem hash. The default puts bundler in each ruby:

    node[:rvm][:global_gems] = [
      { :name => "bundler" }
    ]

## `gems`

A list of gem hashes to be installed into arbitrary RVM rubies and gemsets.
See the `rvm_gem` resource for more details about the options for each gem
hash and target ruby environment. The default is an empty hash.

## `rvmrc`

A hash of system-wide `rvmrc` options. The key is the RVM setting name
(in String or Symbol form) and the value is the desired setting value.
An example used on a build box might be:

    node[:rvm][:rvmrc] = {
      'rvm_gemset_create_on_use_flag' => 1,
      'rvm_trus_rvmrcs_flag'          => 1
    }

The default is an empty hash.

## `root_path`

The path prefix to RVM in a system-wide installation. The default is
`/usr/local/rvm`.

## `group_users`

A list of users that will be added to the `rvm` group. These users
will then be able to manage RVM in a system-wide installation. The default
is an empty list.

# RESOURCES AND PROVIDERS

## rvm_ruby

### Actions

Action    |Description                   |Default
----------|------------------------------|-------
install   |Build and install an RVM ruby. See [RVM rubies/installing](http://rvm.beginrescueend.com/rubies/installing/) for more details. |Yes
remove    |Remove the ruby, source files and optional gemsets/archives. See [RVM rubies/removing](http://rvm.beginrescueend.com/rubies/removing/) for more details. |
uninstall |Just remove the ruby and leave everything else. See [RVM rubies/removing](http://rvm.beginrescueend.com/rubies/removing/) for more details. |

### Attributes

Attribute   |Description |Default value
------------|------------|-------------
ruby_string | **Name attribute:** a fully qualified RVM ruby string that could contain a gemset. See the section *RVM Ruby Strings* for more details. If a gemset is given (for example, `ruby-1.8.7-p330@awesome`), then it will be stripped. |`nil`

### Examples

#### Install Ruby

    rvm_ruby "ree-1.8.7-2011.01" do
      action :install
    end

    rvm_ruby "jruby-1.5.6"

**Note:** the install action is default, so the second example is a more common
usage.

#### Remove Ruby

    rvm_ruby "ree-1.8.7-2011.01" do
      action :remove
    end

**Note:** the RVM documentation mentions that this method is far preferred to
using uninstall since it purges almost everything.

#### Uninstall Ruby

    rvm_ruby "ree-1.8.7-2011.01" do
      action :uninstall
    end

## rvm_default_ruby

This resource sets the default RVM ruby, optionally with gemset. The given
ruby will be installed if it isn't already and a gemset will be created in
none currently exist. If multiple declarations are used then the last executed
one "wins".

### Actions

Action    |Description                   |Default
-------|------------------------------|-------
create |Set the default RVM ruby. See [RVM rubies/default](http://rvm.beginrescueend.com/rubies/default/) for more details. |Yes

### Attributes

Attribute   |Description |Default value
------------|------------|-------------
ruby_string | **Name attribute:** a fully qualified RVM ruby string that could contain a gemset. See the section *RVM Ruby Strings* for more details. If a gemset is given (for example, `ruby-1.8.7-p330@awesome`), then it will be included. |`nil`

### Examples

#### Setting The Default Ruby

    rvm_default_ruby "ree-1.8.7-2011.01" do
      action :create
    end

    rvm_default_ruby "jruby-1.5.6"

**Note:** the create action is default, so the second example is a more common
usage.

## rvm_environment

This resource ensures that the specified RVM ruby is installed and the optional
gemset is created. It is a convenience resource which wraps `rvm_ruby` and
`rvm_gemset` so it can be used as a sort of *über ruby* resource which
parallels the `rvm_default_ruby` resource.

### Actions

Action    |Description                   |Default
-------|------------------------------|-------
create |Installs the specified RVM ruby and gemset. |Yes

### Attributes

Attribute   |Description |Default value
------------|------------|-------------
ruby_string | **Name attribute:** a fully qualified RVM ruby string that could contain a gemset. See the section *RVM Ruby Strings* for more details. If a gemset is given (for example, `ruby-1.8.7-p330@awesome`), then it will be used. |`nil`

### Examples

#### Creating A Passenger Environment In Production

    rvm_environment "ree-1.8.7-2011.01@passenger"

## rvm_gemset
See [RVM gemsets](http://rvm.beginrescueend.com/gemsets/) for more background
concerning gemsets.

### Actions

Action    |Description                   |Default
----------|------------------------------|-------
create   |Creates a new gemset in a given RVM ruby. See [RVM gemsets/creating](http://rvm.beginrescueend.com/gemsets/creating/) for more details. |Yes
update |Update all gems installed to the gemset in a given RVM ruby. |
empty |Remove all gems installed to the gemset in a given RVM ruby. See [RVM gemsets/emptying](http://rvm.beginrescueend.com/gemsets/emptying/) for more details. |
delete    |Delete gemset from the given RVM ruby. See [RVM gemsets/deleting](http://rvm.beginrescueend.com/gemsets/deleting/) for more details. |

### Attributes

Attribute   |Description |Default value
------------|------------|-------------
gemset      | **Name attribute:**  Either a fully qualified RVM ruby string containing a gemset or a bare gemset name. If only the gemset name is given, then the `ruby_string` attribute must be used to indicate which RVM ruby to target. |`nil`
ruby_string | A fully qualified RVM ruby string that should not contain a gemset. See the section *RVM Ruby Strings* for more details. |`nil`

### Examples

#### Creating A Gemset

    rvm_gemset "rails" do
      ruby_string "ruby-1.9.2-p136"
      action      :create
    end

    rvm_gemset "ruby-1.9.2-p136@rails"

**Note:** the create action is default, so the second example is a more common
usage.

#### Updating A Gemset

    rvm_gemset "jruby-1.6.0.RC2@development" do
      action :update
    end

#### Emptying A Gemset

    rvm_gemset "development" do
      ruby_string "jruby-1.6.0.RC2"
      action      :empty
    end

#### Deleting A Gemset

    rvm_gemset "ruby-1.9.2-p136@rails" do
      action :delete
    end

# USAGE

# LICENSE and AUTHOR

Author:: Fletcher Nichol (<fnichol@nichol.ca>)

Contributors:: Bram Swenson (<bram@craniumisajar.com>)

Contributors:: Phil Cohen (http://phlippers.net/)

Copyright:: 2010, 2011, Fletcher Nichol

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.