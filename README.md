# ⚒ Deprecation Toolkit ⚒

## Introduction

Deprecation Toolkit is a gem that helps you get rid of deprecations in your codebase.
Having deprecations in your application usually means that something will break whenever you upgrade third-party dependencies. The sooner the better to fix them!
Fixing all deprecations at once might be tough depending on the size of your app and the number of deprecations. You might have to progressively resolve them while making sure your team doesn't add new ones. This is where this gem comes handy!


## How it works

The Deprecation Toolkit gem works by using a [shitlist approach](https://confreaks.tv/videos/reddotrubyconf2017-shitlist-driven-development-and-other-tricks-for-working-on-large-codebases).
First, the gem records all existing deprecations into `.yml` files. When running a test that have non-recorded deprecations after the initial recording, Deprecation Toolkit triggers a behavior of your choice (by default it raises an error).

## Recording Deprecations

As said above, the Deprecation Toolkit works by using a shitlist approach. You have two ways to record deprecations.
Either set `DeprecationToolkit::Configuration.behavior` to `DeprecationToolkit::Behaviors::Record` (see the Configuration Reference below)
Or run your tests with the `--record-deprecations` flag (or simply the `-r` shortcut)
```sh
rails test <path_to_my_test.rb> -r
```

## Configuration Reference

### 🔨 `#DeprecationToolkit::Configuration#deprecation_path`

You can control where recorded deprecations are saved and read from. By default, deprecations are recorded in the `test/deprecations` folder.

The `deprecation_path` either accepts a string or a proc. The proc is called with the path of the running test file.

```ruby
DeprecationToolkit::Configuration.deprecation_path = 'test/deprecations'
DeprecationToolkit::Configuration.deprecation_path = -> (test_location) do
  if test_location == 'admin_test.rb'
    'test/deprecations/admin'
  else
    'test/deprecations/storefront'
  end
end
```

### 🔨 `#DeprecationToolkit::Configuration#behavior`

Behaviors define what happens when non-recorded deprecations are encountered.

Behaviors are classes that have a `.trigger` class method.

This gem provides 3 behaviors, the default one being `DeprecationToolkit::Behaviors::Raise`.

* `DeprecationToolkit::Behaviors::Raise` will raise either:
  - `DeprecationToolkit::DeprecationIntroduced` error if a new deprecation is introduced.
  - `DeprecationToolkit::DeprecationRemoved` error if a deprecation was removed (compare to the one recorded in the shitlist).
* `DeprecationToolkit::Behaviors::Record` will record deprecations.
* `DeprecationToolkit::Behaviors::Disabled` will do nothing.
  - This is useful if you want to disable this gem for a moment without removing the gem from your Gemfile.

```ruby
DeprecationToolkit::Configuration.behavior = DeprecationToolkit::Behaviors::Record
```

You can also create your own behavior class and perform the logic you want. Your behavior needs to respond to `.trigger`.

```ruby
class StatsdBehavior
  def self.trigger(test, deprecations, recorded_deprecations)
     # Could send some statsd event for example
  end
end

DeprecationToolkit::Configuration.behavior = StatsdBehavior
```

### 🔨 `#DeprecationToolkit::Configuration#allowed_deprecations`

You can ignore some depcreations using `allowed_deprecations`. `allowed_deprecations` accepts an array of Regexp.

Whenever a deprecation matches one of the regex, it is ignored.

```ruby
DeprecationToolkit::Configuration.allowed_deprecations = [/Hello World/]

ActiveSupport::Deprecation.warn('Hello World') # ignored by Deprecation Toolkit
```

### 🔨 `#DeprecationToolkit::Configuration#warnings_treated_as_deprecation`

Most gems don't use `ActiveSupport::Deprecation` to deprecate their code but instead just use `Kernel#warn` to output
a message in the console.

Deprecation Toolkit allows you to configure which warnings should be treated as deprecations in order for you
to keep track of them as if they were regular deprecations.

## License

Deprecation Toolkit is licensed under the [MIT license](LICENSE.txt).
