# Creating and Extending Models

Oxidized supports a growing list of [operating system types](Supported-OS-Types.md). Out of the box, most model implementations collect configuration data. Some implementations also include a conservative set of additional commands that collect basic device information (device make and model, software version, licensing information, ...) which are appended to the configuration as comments.

A user may wish to extend an existing model to collect the output of additional commands. Oxidized offers smart loading of models in order to facilitate this with ease, without the need to introduce changes to the upstream source code.

## Extending an existing model with a new command

The example below can be used to extend the `JunOS` model to collect the output of `show interfaces diagnostics optics` and append the output to the configuration file as a comment. This command retrieves DOM information on pluggable optics present in a `JunOS`-powered chassis.

Create the file `~/.config/oxidized/model/junos.rb` with the following contents:

```ruby
require 'oxidized/model/junos.rb'


class JunOS


    cmd 'show interfaces diagnostics optics' do |cfg|
        comment cfg
    end


end
```

Due to smart loading, the user-supplied `~/.config/oxidized/model/junos.rb` file will take precedence over the model with the same name included in Oxidized. The code then uses `require` to load the included model, and extends the class defined therein with an additional command.

Intuitively, it is also possible to:

* Completely re-define an existing model by creating a file in `~/.config/oxidized/model/` with the same name as an existing model, but not `require`-ing the upstream model file.
* Create a named variation of an existing model, by creating a file with a new name (such as `~/.config/oxidized/model/junos-extra.rb`), Then `require` the original model and extend its base class as in the above example. The named variation can then be specified as an OS type for some devices but not others when defining sources.
* Create a completely new model, with a new name, for a new operating system type.

## Advanced features

The loosely-coupled architecture of Oxidized allows for easy extensibility in more advanced use cases as well.

The example below extends the functionality of the `JunOS` model further to collect `display set` formatted configuration from the device, and utilizes the multi-output functionality of the `git` output to place the returned configuration in a separate file within a git repository.

It is possible to configure the `git` output to create new subdirectories under an existing repository instead of creating new repositories for each new defined output type (the default) by including the following configuration in the `~/.config/oxidized/config` file:

```yaml
output:
    git:
        type_as_directory: true
```

Then, `~/.config/oxidized/model/junos.rb` is adapted as following:

```ruby
require 'oxidized/model/junos.rb'


class JunOS


    cmd 'show interface diagnostic optics' do |cfg|
        comment cfg
    end

    cmd 'show configuration | display set' do |cfg|
        cfg.type = "junos-set"
        cfg.name = "set"
        cfg
    end


end
```

The output of the `show configuration | display set` command is marked with a new arbitrary alternative output type, `junos-set`.  The `git` output will use the output type to create a new subdirectory by the same name. In this subdirectory, the `git` output will create files with the name `<devicename>--set` that will contain the output of this command for each device.