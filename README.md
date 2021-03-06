SLANG
=======
A bridge between SPAN and Clang.

Authors: <br>
- Ronak Chauhan (r.chauhan@somaiya.edu) <br>
- Anshuman Dhuliya (dhuliya@cse.iitb.ac.in)

Summary
--------
The SLANG project interfaces SPAN (Synergistic Program Analyzer) with Clang. Specifically it does the following:

1. Converts Clang's AST to CFG based SPAN IR.
2. Processes SPAN resuts to generate Clang checker reports.

Useful Info
------------
* `grep -R TODO` to get list of things todo.
* `grep -R FIXME` to get list of things to fix.
* `grep -R Author` to get list of authors.

FAQ's
----------

### What is the supported Clang version?

Currently the system is tested to work on Clang 6.0.1 only. We have plans to shift to Clang 7.0.1.

### How to use?

We require that clang/llvm to be built from source. `MY_LLVM_DIR` points to the directory housing the `build` as well as the `llvm` source directory.

Now to use `CFG-plugin/MyDebugChecker.cpp` do the following,

    $ cp CFG-plugin/MyDebugChecker.cpp $MY_LLVM_DIR/llvm/tools/clang/lib/StaticAnalyzer/Checkers/

or you can also create a symbolic link with the same name in the Clang's source, to point to the `cpp` file in this repo (recommended).

Modify `$MY_LLVM_DIR/llvm/tools/clang/lib/StaticAnalyzer/Checkers/CMakeLists.txt` to add the name of the new source file so that `cmake` can pick it up. Add the file name just below  the line that reads `DebugCheckers.cpp`, maintaining the indentation. The file would then read something like this,

    $ cd $MY_LLVM_DIR/llvm/tools/clang/lib/StaticAnalyzer/Checkers/
    $ cat CMakeLists.txt
    ...
      DebugCheckers.cpp
      MyDebugCheckers.cpp
    ...

Modify `$MY_LLVM_DIR/llvm/tools/clang/include/clang/StaticAnalyzer/Checkers/Checkers.td` and add the three line `MyDebugChecker` entry just below the `CFGViewer` entry, as shown below,

    $ cd $MY_LLVM_DIR/llvm/tools/clang/include/clang/StaticAnalyzer/Checkers/
    $ cat Checkers.td
    ...
    def CFGViewer : Checker<"ViewCFG">,
      HelpText<"View Control-Flow Graphs using GraphViz">,
      DescFile<"DebugCheckers.cpp">;
    
    def MyCFGDumper : Checker<"MyDumpCFG">,
      HelpText<"Checker to convert Clang AST to SPAN IR and dump it.">,
      DescFile<"MyDebugCheckers.cpp">;
    ...


Now go to the `$MY_LLVM_DIR/build` directory and build the system using `make` or `ninja` (which ever you have used to build clang/llvm system).

Once done you can use the checker as any other checker in the system. The invocation name of the checker is `debug.MyDumpCFG`.


Misc Info
--------------------------------

### Why does CMake use full paths, or can I copy my build tree?

CMake uses full paths because:

* configured header files may have full paths in them, and moving those files without re-configuring would cause upredictable behavior.

* because cmake supports out of source builds, if custom commands used relative paths to the source tree, they would not work when they are run in the build tree because the current directory would be incorrect.

* on Unix systems rpaths might be built into executables so they can find shared libraries at run time. If the build tree is moved old executables may use the old shared libraries, and not the new ones.

#### Can the build tree be copied or moved?

The short answer is NO. The reason is because full paths are used in CMake, see above. The main problem is that cmake would need to detect when the binary tree has been moved and rerun. Often when people want to move a binary tree it is so that they can distribute it to other users who may not have cmake in which case this would not work even if cmake would detect the move.

The workaround is to create a new build tree without copying or moving the old one.

Resource = [kitware-faq](https://gitlab.kitware.com/cmake/community/wikis/FAQ#why-does-cmake-use-full-paths-or-can-i-copy-my-build-tree)

