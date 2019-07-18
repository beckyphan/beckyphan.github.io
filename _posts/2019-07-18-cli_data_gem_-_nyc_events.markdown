---
layout: post
title:      "CLI Data Gem - NYC Events"
date:       2019-07-18 17:22:21 +0000
permalink:  cli_data_gem_-_nyc_events
---


> Places to go,
> People to see,
> There are a million things to do
> in NYC.
> 

For my CLI data gem project, I'll be scraping nyc.com/events to populate a list of events, narrowed down to events occurring today.


The general classes I'll use for my project will be:
- Scraper class will scrape the events page to collect page details & sort out the data 
- Event class will create objects that will contain details of the events, including name, date, location, description, etc.
- List class will create a cart category that users can add events to in order to view a list of events they've collected/are interested in
- CLI file will contain the output & get user input 

The general scraping portion is relatively simple using Nokogiri and open-uri. Inspecting the site showed that all the events are under the class event records with each line item containing an event having an attribute of itemtype='http://schema.org/Event'. It took me a while to figure out how to select for an attribute, but alas, I figured it out!

![](http://imgur.com/RXKUPe4)

Because NYC has a million events going on at any one given time, initially I was going for a weekly list, but upon seeing how many events are happening in a single day... it's a toughie to list them all out. 

Additionally, I noticed that running the code in repl.it only listed a handful of events. Because the web page that I am scraping from uses infinite scrolling, I need to figure out how to scroll allllll the way down so that I can scrape allllll of the events. Or at least how to scrape more than a handful.

When trying to research it, most users were talking about use of Scrapy and BeautifulSoup. Since I was not so familiar, I gleaned what I could. The dev tools > Network > XHR showed me the webpage URL of the new 'pages' that populated as I scrolled down. Thus, I incorporated a prompt for users to request more events, which would trigger scraping of that new page url to return additional events and keep count of which page has been scraped.

All events are added to an Events array. Thankfully, when scraping for more -- it doesn't rescrape the previous pages so there was no need to check the Events array for duplicates of the first 20 or so events previously displayed.

Once the application was working properly via the console or via repl.it, I needed to get the actual application up to code with all its other files and setting up the correct environments for the application to run.

- README.md => contains the instructions & all details regarding the application, along with installation instructions

- bin/console => typing in bin/console opens up IRB as a test environment
- bin/nyc_events.rb => running this files starts up the program
- bin/setup => provides the setup for the program, runs bundle install to install any missing gems required

- lib/nyc_events => has all my ruby files to run the application (CLI, event, list, scraper, etc)

- Gemfile => had to require 'nokogiri' and  'open-uri' here for the program to run properly
- Gemfile.lock => seems to be updated after I run 'bundle install' 
- License.txt => Used MIT License
- nyc_events.gemspec => added my other dependencies here, including nokogiri. Learned that OpenUri is part of the Ruby standard library, so you only need to require it if you want to use it in your code aka no gem required/no dependency on a gem here
- Rakefile => allows for running files as a suite using a single command; for my gem, I didn't make any changes to what was in the template.

There's definitely more for me to understand regarding uses for these files, but as of now, it looks like it has everything it needs for the gem to run and for the event scraper to be called by running "bin/nyc_events.rb"

Upon playing with the CLI more, I realized that the description of the events are cut off & the read more leads to a page with the more information. Since the user will need to visit the event's site in order to read more or to purchase tickets, etc., I scrapped the value within the href attribute to display to the user.

I was reading about naming files/gems as well and saw the importance of using namespaces, especially since the classes I have (Event, List, Scraper, etc.) are pretty generic/common so I went through and renamed my classes to utilize namespaces...

additionally, I ran into an issue while testing because some events did not have links to more info, thus was causing the program to break and had to mitigate this by incorporating a ternary operator to collect an event link when available.

and I think that's it? 

Ta-Da! A CLI application that scrapes the millions of events happening in NYC. 
