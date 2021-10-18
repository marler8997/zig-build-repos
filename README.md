# zig-build-repos

Enables `build.zig` files to depend on git repositories.

# How to use

For now the recommendation to use this repository is to copy the `GitRepoStep.zig` file into your local repository alongside youre `build.zig` file.  Once you do this, it will be available for import by adding:

```zig
const GitRepoStep = @import("GitRepoStep.zig");
```

Here is an example of how to create and use a `GitRepoStep`:

```zig
const zigwin32_repo = GitRepoStep.create(b, .{
    .url = "https://github.com/marlersoft/zigwin32",
    .branch = "10.3.16-preview",
    .sha = "74942bfb350f38f18b39db47c97c1274f6c418b4",
});

const exe = b.addExecutable("foo", "foo.zig");
// more exe setup

exe.step.dependOn(&zigwin32_repo.step);
exe.addPackagePath("win32", try std.fs.path.join(b.allocator, &[_}[]const u8 {
    zigwin32_repo.getPath(&exe.step), // getPath will ensure step dependencies are correct
    "win32.zig",
}));
```

The code above will add the `zigwin32` git repository as a dependency of the executable `foo` that was added.  During the build, if the git repository step is required by the current target, `GitRepoStep` will check if the repository is downloaded, if not then by default it will report an error that the repository is missing and provide the user with the git command to clone it.

```sh
$ zig build
Error: git repository '/home/user/my-foo-project/dep/zigwin32' does not exist
       Use -Dfetch to download it automatically, or run the following to clone it:
       git clone https://github.com/marlersoft/zigwin32 /home/user/my-foo-project/dep/zigwin32 && git -C /home/user/my-foo-project/dep/zigwin32 checkout 74942bfb350f38f18b39db47c97c1274f6c418b4 -b fordep
```

If the build option `-Dfetch` was given, it will automatically clone any missing repositories that are required by the current build.

```sh
$ zig build -Dfetch
info: [RUN] "git" "clone" "https://github.com/marlersoft/zigwin32" "/home/user/my-foo-project/dep/zigwin32"
Cloning into '/home/user/my-foo-project/dep/zigwin32'...
remote: Enumerating objects: 324, done.        
remote: Counting objects: 100% (324/324), done.        
remote: Compressing objects: 100% (143/143), done.        
remote: Total 324 (delta 201), reused 289 (delta 176), pack-reused 0        
Receiving objects: 100% (324/324), 175.07 KiB | 1.18 MiB/s, done.
Resolving deltas: 100% (201/201), done.
info: [RUN] "git" "-C" "/home/user/my-foo-project/dep/zigwin32" "checkout" "74942bfb350f38f18b39db47c97c1274f6c418b4" "-b" "fordep"
Switched to a new branch 'fordep'

```

If the repository is already downloaded and the SHA does not match what is expected, by default it will print a warning to the user with the expected/actual SHA of the repository.

```sh
$ zig build
warning: repository 'zigwin32' sha does not match
expected: 74942bfb350f38f18b39db47c97c1274f6c418b4
actual  : ea3ee83dd816f815cedcb67e4896ade701038b90

```

This behavior can be ignored or become a fatal error with the `sha_check` option passed to `GitRepoStep`.

Also note that `GitRepoStep.zig` can be used to download any git repository, not just ones containing zig packages.  For more real-world examples take a look at https://github.com/marler8997/ziget/blob/master/build.zig

# Real World Examples

* https://github.com/marler8997/ziget/blob/master/build.zig
* https://github.com/marler8997/zigup/blob/master/build.zig
* https://github.com/marler8997/windows-remote-control/blob/master/build.zig
* https://github.com/marlersoft/zigwin32gen/blob/main/build.zig
