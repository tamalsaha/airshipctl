airshipctl
==========

How to do plugins while also having a single binary
---------------------------------------------------

### 1. Go's builtin [plugin][plugin] package with a bin packer (such as [go-bindata][bindata])
#### Implementation
  1. Create an interface for creating cobra.Commands
  2. Create a plugin that satisfies the interface
  3. Compile the plugin to a .so file
  4. Compile airshipctl to a single binary, packing in the .so files
  5. Distribute the binary
  6. When running airshipctl, unpack the plugins, load them, then continue
     execution
#### Pros
  - There's a single executable binary
#### Cons
  - Go's plugin library is a bit janky, and plugins may not work if they aren't
    compiled with the exact same packages as the executable
  - There's a single executable binary, but it depends on multiple shared
    objects to function
  - Doesn't work with static compilation

### 2. Lots of binaries and use [kubectl's strategy][kubectl-plugin]
#### Implementation
  1. Compile airshipctl to a single executable binary with no subcommands
  2. At runtime, search a designated path (e.g. $PATH) for executables named
     airshipctl-$PLUGIN_NAME, and pull them in as subcommands
#### Pros
  - We can pull in any pre-built executable just by renaming it and putting it
    in the right place
  - Plugins are managed indepenedently airshipctl, and can be written in any
    language
  - airshipctl can be statically compiled
#### Cons
  - There's multiple binaries
  - The plugins can't take advantage of package code provided by airshipctl

### 3. Separate airshipctl into multiple projects
#### Implementation
  1. Separate airshipctl into 2 projects: airshipctl-lib and airshipctl-cmd
  2. Put all of the shared code into airshipctl-lib
  3. Create commands in airshipctl-cmd that use the code in airshipctl-lib
  4. Compile airshipctl-cmd to get a single binary. I'm not sure how to do this
     with 2+ plugins yet
#### Pros
  - Single binary
  - Plugins are ridiculously simple, leveraging airshipctl-lib and creating
    small subcommands
#### Cons
  - This might not even work - I have no idea how to compile multiple plugins
    together
  - Plugins that strive to be larger subcommands will suffer because they are
    expected to live in a cmd directory
  - This creates TWO versioned projects. We'd need to assure that
    airshipctl-cmd version X is compatible with airshipctl-lib version Y on top
    of the other dependencies

### 4. Containerize
#### Implementation
  1. Wrap everything up into a container and distribute that
#### Pros
  - There's one entity being distributed
  - We get our conglomerate of applications
#### Cons
  - It's weird. No one does this
  - The plugins can't take advantage of package code provided by airshipctl


[plugin]: https://golang.org/pkg/plugin/
[bindata]: https://github.com/jteeuwen/go-bindata
[kubectl-plugin]: https://github.com/kubernetes/kubernetes/blob/master/pkg/kubectl/cmd/cmd.go#L327-L408
