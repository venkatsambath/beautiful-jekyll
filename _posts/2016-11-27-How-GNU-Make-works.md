---
layout: post
title: How GNU make works?
---

When make is invoked with no arguments, it looks in the current directory for a file named GNUmakefile, makefile or Makefile, and attempts to build the first target it contains, called the default target. If the default target is up to date—meaning that it exists, that all its prerequisites are up to date, and that none of its prerequisites has been modified more recently than it has—make’s job is done. Otherwise, it attempts to generate the default target from its prerequisites by executing its command script. Like the definition of up to date, this process is recursive: for each prerequisite which is not up to date, make searches for a rule having that prerequisite as a target, and starts the whole process again. This continues until the default target is up to date or until an error occurs.

It follows from the above description that a target having no prerequisites is up to date if and only if it corresponds to a file on the filesystem. Therefore, a target corresponding to a non-existent file is never up to date, and can be used to force a command script to be executed unconditionally. Such targets are called phony targets

