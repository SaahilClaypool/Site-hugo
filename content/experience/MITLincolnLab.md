---
date: 2017-09-26T08:08:14+02:00
draft: false
title: MIT Lincoln Laboratory
tags: 
- work
- 2016
---

# MIT Lincoln Laboratory software Internship 2016

I worked [MIT Lincoln Laboratory](https://ll.mit.edu/about/about.html) the summer of 2016 in a group working for the FAA. Here I created a visualization tool to monitor a distributed system of computers as a graph network. This tool helped visualize when a node in the graph failed and what effect this had on the rest of the system. 

## Details

I worked at [MIT Lincoln Laboratory](https://ll.mit.edu/about/about.html) the summer of 2016. Lincoln is a federally funded research and development center (FFRDC) that is associated to MIT. I worked in a group hat was funded by the the FAA (Federally Aviation Association), and as a result most of their work had to do with planes and weather research. 

I specifically worked on a project that provides weather data for all of the planes in the U.S. The team responsible aggregates data from thousands of weather data and runs this data through a computational computer cluster to create weather visualizations for flight routing. This is used constantly throughout the country to rout planes through difficult weather patterns, and thus it is *very* important this weather system does not go down. So, this team in Lincoln had a large number of people who's' sole responsibility was to monitor this system, making sure everything was working at any given point in the day. 

To monitor this system, the team at Lincoln used a graph network visualization tool. Each node in the graph was a single computer, and the edges or connections represented the flow of data from one computer to another. While this tool worked for the monitoring team, it was very hard to maintain as it required someone to *manually* lay out this graph network in the python code for the tool every time the network changed. This quickly was becoming infeasible as there were upwards of 1,000 computers in this network. This is where my job came into play. 

My job was to evaluate a trial license for a graph visualization tool called [Tom Sawyer Visualization](https://www.tomsawyer.com/products/visualization/) by creating a new, more maintainable, monitoring tool for the monitoring team. This tool could read a file from a format such as JSON or a CSV and dynamically create a fitting graph structure. And, this tool came with HTML5 bindings so it could be hosted and used remotely at any time which would be very flexible. 

To summarize the actual work, I spent most of my time parsing configuration files and creating a script that would dynamically determine which nodes were sending data to other nodes. Then, this would check the status of these nodes from a server. All of this connection and status data would then be written to a JSON file. Another program, with the Tom Sawyer Packages, would then read this file and host the graph visualization in an html5 canvas for the monitors to see. All of this code was written in java, except for a small portion of javascript in the frontend to get the Tom Sawyer frontend talking to the Java server it ran in. I then spent a good portion of seeing what other capabilities the Tom Sawyer software had, such as rearranging nodes, filtering the tree to look at just smaller portions, and generally determining how flexible the tool was. All of this wrapped up with a presentation to the group where I gave my 'review' of the visualization software and demoed my tool. 

All in all, the internship was a great experience. I was exposed to everything from legacy python 2 code, Apache java servers, TCP networks, and real, non computer scientist, human interaction. 