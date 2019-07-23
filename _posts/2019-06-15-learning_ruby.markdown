---
layout: post
title:      "learning ruby"
date:       2019-06-15 17:01:10 -0400
permalink:  learning_ruby
---


> the coursework starts off teaching ruby
> its variables, methods, falsey/truthy.
> then we dive into procedural,
> obj-oriented, relationships, architectual,
> metaprogramming; hey, we're all scraping by...

Who doesn't love a good limerick?

The past 4 weeks has been a whirlwind and I feel like I've learned a lot and yet I know I've hardly grazed the surface.

There are a few topics I need to talk aloud to cement, including:
* hashes
* object relationships (belongs to, has many)
* methods for metaprogramming
* scraping

**Hashes**
Hashes are key-value(s) pairs similar to an unordered dictionary. 
Like python's dictionaries, they are written with curly braces {}.
Keys stay constant, and each named key is a symbol, where symbols are written like :symbol.
BUT, symbols can be written with different notations 
i.e. {key: "value"} or {:key => "value"}
keys can also be strings and assigned values with the rocket =>
i.e. {"key" => "value"}
```
hash = {key: "value"}
=> {:key=>"value"}

#refer to a value by calling its key in brackets, similar to calling an array's index
hash[:key]
 => "value"
```

**Object Relationships**
Belongs To & Has Many
If an object belongs to another object, it should be able to pull information from its related object.
e.g. a song belongs to an artist, so we would expect when we look up a song's artist, it'll reference and return the artist object.

Has Many Objects Through
If an object is distantly related to another object, they should all still be able to pull information from its related objects.
e.g. a museum has many objects and those objects have a curator, so a museum should be able to list all its objects and we could also list all the curators that have donated to the museum. 

Initially, I didn't get that I was throwing in objects as arguments and I was trying to throw names or artists in or variables that didn't yet exist (I'm v wary of creating new instances of objects that I lose into the abyss because I don't save them to some variable), but with some practice, I think I'm getting the hang of this.


**Metaprogramming**
I understand the purpose and flexibility of keyword arguments, but it's not intuitive that the variables are assigned the same way.

i.e.
Where order of arguments matter:
```
  def initialize(user_name, name, age, location)
    @user_name = user_name
    @name = name
    @location = location
    @age = age
  end
```
Where order doesn't matter bc using keyword arguments:
```
  def initialize(user_name:, name:, age:, location:)
    @user_name = user_name
    @name = name
    @location = location
    @age = age
  end
```

I redid an earlier lab and changed the objects to initialize with keyword arguments... I had to update the tests to also accept keyword arguments so I hope I didn't break anything... the tests passed eventually but hopefully because I did it correctly and not because I accidentally botched something in rspec.

**Scraping**
Steps to Scraping Data from Websites

1. gem install nokogiri // installs the nokogiri gem that allows us to parse HTML by treating HTML string as nested node
2. in a new ruby file, we need to:
* require 'nokogiri' (gem that allows parsing HTML)
*  require 'open-uri' (module that allows HTTP requests)
3. open the URL and save it to a variable to work with
*  html = open("https://flatironschool.com/")
4. use nokogiri to convert the HTML to a nodeset and save to a variable
* doc = Nokogiri::HTML(html)

from there, we use .css selectors and pull .text that we want. 
Note-to-Self: definitely need to put this into practice & fine-tune my steps/add details/correct as needed

TTFN!

