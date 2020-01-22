---
layout: post
title:      "CLI Data Project Creation"
date:       2020-01-22 22:01:02 +0000
permalink:  cli_data_project_creation
---



This was not my first coding project but it was the first public one. I was aware it needed to be evaluated and perhaps used by others. The main goal became clean code rather than just to make it work.

I found it difficult to follow a top-down approach. My process was instead to go directly to the scraping details/issues and see what I could achieve and what were the technical limitations.

Once I figure that out I passed to design and create the Classes to store the scraped information.

Only when I felt the data was organised in a nice herarchy I looked at the interface and thought from the end-user perspective.

At this stage I went back and forth between Interface, Scraping Classes and Data Classes so to shape the latter to how the design of the CLI interface was coming along.
I also started worrying about how the different Classes where interacting with each other, mainly driven from the interface requirements.

I would say I used a down-top-down approach :)

All the way I tried to DRY. 
I realised how my attempt to do something simple and elegant got out of hand. The growing features are an animal difficult to tame with simplicity. 
All the initial blue prints continuosly get scraped and revisited by a maturing project that the more progresses, the more it seems gaining possible developments, completion getting further. 
Suddenly the focus on elegance vanishes and the main scope becomes again just to make the next "great" idea work.

Nevertheless the creative process was very enjoyable, and if I didn't have a deadline I would still be coding away on a CLI app... that noone needs hahaha.

A good portion of the learning process has been for me to clarify how the package manager works behind the hood so to clarify what was the essential for the project to be installed smoothly with all the libraries. 
I feel that worrying about that, rather than focusing only on the app coding puzzles, is making me a more complete and mature coder. 
