#!/usr/bin/env python
import os
import sys
from glob import glob
from pathlib import Path

# Neccessary to have our own build options without errors
SAVED_ARGUMENTS = ARGUMENTS.copy()
ARGUMENTS.pop('intermediate_delete', True)

env = SConscript("addons/godot-av1/deps/godot-cpp/SConstruct")

ARGUMENTS = SAVED_ARGUMENTS

# Custom options and profile flags.
customs = ["custom.py"]
profile = ARGUMENTS.get("profile", "")
if profile:
    if os.path.isfile(profile):
        customs.append(profile)
    elif os.path.isfile(profile + ".py"):
        customs.append(profile + ".py")
opts = Variables(customs, ARGUMENTS)

opts.Add(
    BoolVariable("intermediate_delete", "Enables automatically deleting unassociated intermediate binary files.", True)
)

opts.Update(env)
Help(opts.GenerateHelpText(env))

def GlobRecursive(pattern, node='.'):
    import SCons
    results = []
    for f in Glob(str(node) + '/*', source=True):
        if type(f) is SCons.Node.FS.Dir:
            results += GlobRecursive(pattern, f)
    results += Glob(str(node) + '/' + pattern, source=True)
    return results

# For the reference:
# - CCFLAGS are compilation flags shared between C and C++
# - CFLAGS are for C-specific compilation flags
# - CXXFLAGS are for C++-specific compilation flags
# - CPPFLAGS are for pre-processor flags
# - CPPDEFINES are for pre-processor defines
# - LINKFLAGS are for linking flags

# tweak this if you want to use different folders, or more folders, to store your source code in.
env.Append(CPPPATH=["addons/godot-av1/src/"])
sources = GlobRecursive("*.cpp", "addons/godot-av1/src")

# Remove unassociated intermediate binary files if allowed, usually the result of a renamed or deleted source file
if env["intermediate_delete"]:
    def remove_extension(file : str):
        if file.find(".") == -1: return file
        return file[:file.rindex(".")]

    found_one = False
    for obj_file in [file[:-len(".os")] for file in glob("addons/godot-av1/src/*.os", recursive=True)]:
        found = False
        for source_file in sources:
            if remove_extension(str(source_file)) == obj_file:
                found = True
                break
        if not found:
            if not found_one:
                found_one = True
                print("Unassociated intermediate files found...")
            print("Removing "+obj_file+".os")
            os.remove(obj_file+".os")

if env["platform"] == "macos":
    library = env.SharedLibrary(
        "addons/godot-av1/bin/godot-av1/libgodot-av1.{}.{}.framework/libgodot-av1.{}.{}".format(
            env["platform"], env["target"], env["platform"], env["target"]
        ),
        source=sources,
    )
else:
    suffix = ".{}.{}.{}".format(env["platform"], env["target"], env["arch"])
    library = env.SharedLibrary(
        "addons/godot-av1/bin/godot-av1/libgodot-av1{}{}".format(suffix, env["SHLIBSUFFIX"]),
        source=sources,
    )

Default(library)