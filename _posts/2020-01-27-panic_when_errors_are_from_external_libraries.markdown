---
layout: post
title:      "PANIC: When Errors are From External Libraries"
date:       2020-01-27 21:44:25 +0000
permalink:  panic_when_errors_are_from_external_libraries
---


#### I usually don't fear errors. 
I don't fear errors when they come from my own code. Solving them is like the satisfaction of removing a little stone from your shoe, or eventually inserting that USB stick when you understand it was the other way around.
You know you can solve it, you know the field where the battle will happen, you know that victory is near and it will be satisfying. 
Read that error message, and the pleasure begins, your brain already identifies many possible issues, you feel a genius, you are James Bond, Sherlock Holmes, Dr House... nothing can stop you! 
..a little exploration and BOOM! those green fonts will be flowing in your terminal. 

But today after three months of victorius tests, and project creation I feel beaten. stuck in the most innocuous SQL test.

#### What is different? 

This time the error doesn't come from my code, it doesn't even come from the Rspec Test, it comes from an external library.
No more stone in the shoe or stuck USB stick, It is like being dropped in the middle of a foreign ostile city without knowing the local language.

Even when the problem is from the tests, you receive an expect/received clear statement of what is wrong so the path to the solution is always quite a structured process.

What I get this time instead is the following:

```
querying the bears table
  selects all of the female bears and returns their name and age (FAILED - 1)

Failures:

  1) querying the bears table selects all of the female bears and returns their name and age
     Failure/Error: expect(@db.execute(selects_all_female_bears_return_name_and_age)).to eq([["Tabitha", 6],["Melissa", 13], ["Wendy", 6]])
     
     SQLite3::SQLException:
       near "Write": syntax error
     # /Users/sergio/.rvm/gems/ruby-2.6.1/gems/sqlite3-1.3.13/lib/sqlite3/database.rb:91:in `initialize'
     # /Users/sergio/.rvm/gems/ruby-2.6.1/gems/sqlite3-1.3.13/lib/sqlite3/database.rb:91:in `new'
     # /Users/sergio/.rvm/gems/ruby-2.6.1/gems/sqlite3-1.3.13/lib/sqlite3/database.rb:91:in `prepare'
     # /Users/sergio/.rvm/gems/ruby-2.6.1/gems/sqlite3-1.3.13/lib/sqlite3/database.rb:137:in `execute'
     # ./spec/select_spec.rb:14:in `block (2 levels) in <top (required)>'
```


There isn't an output that mismatches what the test expects, instead it seems there is an error occuring inside the sqlite3 library.

What does that `"near "Write": syntax error"` even mean??

#### My Process

First thing I try to replicate what the test is supposedly trying to gather from my table (at this point two files - create.sql and insert.sql - were created from me, in previous tests, to initialize the table from .sql files during the tests)
```
sqlite> select * from bears;

sqlite> SELECT name, age FROM bears WHERE gender = 'female';
name        age       
----------  ----------
Tabitha     6         
Melissa     13        
Wendy       6         
sqlite> 
```

OK, my table is fine, why is it that the test doesn't get the expected result but a `SQLite3::SQLException` instead?
Reading the error origins I realise I have to open that `.../sqlite3/database.rb` at line 137 to figure out what was going wrong.

And here the panic started.. oh dear... where they still writing in ruby in there? things were very complicated to understand and a lot of never seen trickery was going on in that code... I would have never been able to figure out what was going wrong.. I could not even understand how the arguments were given in that #execute that seems to trigger the exception in the test... what is this`def execute sql, bind_vars = [], *args, &block` !?!?

line 137 leads me to another function, #prepare:
```
def prepare sql
      stmt = SQLite3::Statement.new( self, sql )
      return stmt unless block_given?

      begin
        yield stmt
      ensure
        stmt.close unless stmt.closed?
      end
    end
```
After a little look inside the `.../sqlite3/statement.rb` and realized there was not even an #initialize that supposedly was creating the problem I gave up.

Nevertheless I understood there was an initialization failing and that #execute is not some sort of metaprogramming that from the string 'selects_all_female_bears_return_name_and_age' executes an SQL query on my table, as in the first panic I thought... 
inside the code of the sqlite3 library the subject was a string with the query itself, something like "select * from table where a=? and b=?" that was the input of #execute.. not some sort of string!

Wait a moment !  'selects_all_female_bears_return_name_and_age' is not even a string.. is given as a method!

Oh my.. now I understand, reading above I see that this tests is not even testing a database that comes from my insert.sql, they are actually only interested in testing some function that I am supposed to write.

And a little look through the file I find the inciminated function
```
def selects_all_female_bears_return_name_and_age
  "Write your SQL query here"
end
```
WOW! I bet that "Write" is the same mysterious one appearing in the error, I susbstitute the first word of the string, run the test, and yes that's it.

I simply didn't read the README and got the test wrong, I need just to put some simple SQL query string as return value in that function.

I do it and this time a juicy understandable failure from the Rspec Test iteself appears:

```
expected: [["Tabitha", 6], ["Melissa", 13], ["Wendy", 6]]
            got: []
```

I need to put the gender as 'F' and not 'female'... ahhh this is satisfaction :)

I feel I complicated things, but it was a useful lesson.


