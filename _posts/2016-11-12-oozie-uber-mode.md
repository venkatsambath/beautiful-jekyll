---
layout: post
title: What is Ubermode
---


Normally when running a MapReduce Job, Yarn allocates a container and starts a JVM in that container to run the MR Application Master. The MR AM then gets another container from Yarn for each Map and Reduce task and starts a JVM in those to run the Map and Reduce task. In the case of the Oozie Launcher Job, you end up with the MR AM and one extra container for the single Map task that’s running the job (in this case, it was a Java Action, so it was running their Java Main class). This extra container and JVM adds additional overhead, so MapReduce has a feature called Uber Mode where the map task can be run inside of the MR AM. In theory, this sounds perfect for the Oozie Launcher Job. However, we’ve run into a number of tricky edge cases due to it. Typically, these are issues related to things leaking between the MR AM and the Launcher Job’s Map task, but there have been other issues like logging oddities and like this one where a native library wasn’t loaded.

CDH5.8.0 disables uber mode
