# 1. Quering

## Hands-on with RediSearch

- First, find the book The Inner Reaches of Outer Space:

```bash
 FT.SEARCH books-idx "@isbn:{9781577312093}"
 ```

- Now find the author Edward John Moreton Drax Plunkett Baron Dunsany, whose ID is 690:

```bash
FT.SEARCH authors-idx "@author_id:{690}"
```

- Next, find books by Edward John Moreton Drax Plunkett Baron Dunsany:

```bash
FT.SEARCH books-idx "@author_ids:{690}" RETURN 1 title
```

**NOTE:**  You can use the RETURN option to return only some fields.

Finally, find books that have the "Fantasy" category:

```bash
FT.SEARCH books-idx "@categories:{Fantasy}"
```

## Working with numbers

- Try finding the titles of books with an average rating from 4.5 through 5:

```bash
FT.SEARCH books-idx "@average_rating:[4.5 5]" RETURN 1 title
```

- Now find the titles of books with an average rating from 0 through 1:

```bash
FT.SEARCH books-idx "@average_rating:[0 1]" RETURN 1 title
```

- Next, find the titles of books having an average rating of at least 4 and being published on or after 2015:

```bash
FT.SEARCH books-idx "@average_rating:[4 +inf] @published_year:[2015 +inf]" RETURN 1 title
```

- Finally, find the titles of books with an average rating of at most 3 published before 2000:

```bash
FT.SEARCH books-idx "@average_rating:[-inf 3] @published_year:[-inf (2000]" RETURN 1 title
```

## Working with Dates and Times

- Try finding users who have logged in on or after 1:25pm UTC on December 11, 2020:

```bash
 FT.SEARCH users-idx "@last_login:[1607693100 +inf]"
```

- Now try to find users whose last login was prior to 1:25pm UTC on December 11, 2020:

```bash
FT.SEARCH users-idx "@last_login:[-inf (1607693100]"
```

## Boolean Logic

Many of the queries in this section use full-text search because this type of query makes boolean logic simple to illustrate.

Queries that don’t use a specific field default to full-text search using all TEXT fields. So, consider again the following query, taken from the last video:

```bash
FT.SEARCH books-idx "dogs|cats"
```

This query searches for dogs OR cats in all TEXT fields in the index.

For now, don’t worry about exactly how full-text search works. Focus on the boolean logic, and we’ll tell you all about full-text search in the next section.

## Combining Multiple Fields

You can also use boolean logic to combine terms from multiple fields.

Here, we use a boolean AND to find books with the author “rowling” that have “goblet” in the title.

```bash
FT.SEARCH books-idx "@authors:rowling @title:goblet"
```

Just like when you use a boolean OR for terms in a single field, you connect two fields with a boolean OR by joining them with a pipe. In this query, we find books by the author “rowling” or that have “potter” in the title.

```bash
FT.SEARCH books-idx "@authors:rowling | @title:potter"
```

And same as with searching for terms in a field, use the dash symbol for a NOT query between fields. In this query, I'm looking for books by Tolkien whose titles do not include the word "ring".

```bash
FT.SEARCH books-idx "@authors:tolkien -@title:ring"
```

## Sorting Results

- Try finding Juvenile Fiction books sorted by the year they were published:

```bash
FT.SEARCH books-idx "@categories:{Juvenile Fiction}" SORTBY published_year
```

- Now try finding books with an average rating between 4.9 and 5 inclusive, sorted by average rating in descending order:

```bash
FT.SEARCH books-idx "@average_rating:[4.9 5]" SORTBY average_rating DESC
```

> ### Sorting by Multiple Fields
> #### The SORTBY option to FT.SEARCH allows you to sort by only one field per query. However, as you’ll see when we talk about aggregations, you can sort an aggregation query by more than one field.


## Limiting Results (Pagination)

- Try searching for books written by Ursula K. Le Guin, ordered by publication year, and limiting the query to the first 3 books published.

```bash
FT.SEARCH books-idx "@authors:Ursula K. Le Guin" SORTBY "published_year" LIMIT 0 3
```

- Next, try some offset and limit pagination. Starting at offset 100, get the next 100 books from the Fiction category published on or after the year 2000.

```bash
FT.SEARCH books-idx "@published_year:[2000 +inf]" LIMIT 100 100
```

# Let's Practice !

- Try getting Stephen King books published between 1980 and 1990, inclusive.

```bash
FT.SEARCH books-idx "@authors:'Stephen King' @published_year:[1980 1990]"
```

- Now try finding books with the Philosophy category, published on or before 1975, written by anyone other than Arthur Koestler.

```bash
FT.SEARCH books-idx "@categories:{Philosophy} @published_year:[-inf 1975] -@authors:'Arthur Koestler'"
```

>  **Note:** We use curly braces around “Philosophy” in this query because “categories” is a TAG field. As you may recall from our section on exact-string matches, querying TAG fields requires curly braces.

- Finally, try finding books written by Aruthur Koestler OR Michel Foucault:

```bash
FT.SEARCH books-idx "@authors:'Arthur Koestler' | @authors:'Michel Foucault'"
```

- The same query, but this time with results sorted in descending order by published year:

```bash
FT.SEARCH books-idx "@authors:'Arthur Koestler' | @authors:'Michel Foucault'" SORTBY published_year DESC
```

---

# 2. Full-Text Search

Queries also compare your terms against descriptions. Try searching for unicorns with this query:

```bash
FT.SEARCH books-idx unicorns
```

Notice that the results have “unicorn” in their descriptions.

## Prefix matching

You can combine a normal full-text term with a prefix term. Try searching for matches with the term “atwood” and the prefix “hand”:

```bash
FT.SEARCH books-idx "atwood hand*"
```

You can also use multiple prefix terms in a single query. Try searching for “agat* orie*” -- you should find Murder on the Orient Express.

```bash
FT.SEARCH books-idx "agat* orie*"
```

## Boolean logic

Try finding books about dragons that are not also about wizards or magicians!

```bash
FT.SEARCH books-idx "dragons -wizard -magician"
```

## field-specific searches

Now, try a full-text search for “mars” across all TEXT fields with a full-text search for “heinlein” in only the authors field:

```bash
FT.SEARCH books-idx "mars @authors:heinlein"
```

## sorting

Try sorting all books that mention the prefix “crypto” sorted by publication year.

```bash
FT.SEARCH books-idx crypto* sortby published_year
```

## limiting  

Finally, get the first book in order of publication year that mentions “murder”:

```bash
FT.SEARCH books-idx murder sortby published_year limit 0 1
````

## Highlighting & Summarization 

This query returns a maximum of three (which is also the default) "fragments" of twenty-five words each for matches of the term "agamemnon":

```bash
FT.SEARCH books-idx agamemnon SUMMARIZE FIELDS 1 description FRAGS 3 LEN 25
```

> **Note!** : In this context, the matching text is often called a “hit.”

You can combine HIGHLIGHT and SUMMARIZE together to highlight hits in a field and summarize the text returned around each hit.

```bash
FT.SEARCH books-idx agamemnon SUMMARIZE FIELDS 1 description FRAGS 3 LEN 25 HIGHLIGHT
```

- ## Practice

Search for the term “illusion” and highlight any matches:

```bash
FT.SEARCH books-idx illusion highlight
```

Now search for “shield,” highlighting any matches, and summarizing the description field with a max fragments of 1 and length 20.

```bash
FT.SEARCH books-idx shield HIGHLIGHT SUMMARIZE FIELDS 1 description FRAGS 1 LEN 20
```

# 2. Aggregation

FT.SEARCH works perfectly well to count query results, but you can also use the FT.AGGREGATE command to count items in a query.

Here’s an aggregation query that finds the same number of items as the FT.SEARCH query we just looked at:

```bash
FT.AGGREGATE books-idx * GROUPBY 0 REDUCE COUNT 0 AS total
```

This query introduces us to most of the concepts we’ll talk about in this unit. Let’s go through the steps briefly.

## Groupping data

Try grouping book checkouts in the checkouts-idx index by the checkout date.

```bash
FT.AGGREGATE checkouts-idx * GROUPBY 1 @checkout_date
```

Now try a search for “python” in the books-idx index grouped by the “categories” field.

```bash
FT.AGGREGATE books-idx python GROUPBY 1 @categories
```

## Sorting

Try finding all users in the users-idx index, grouped by last login date and last name, and then sorted by last name.

```bash
FT.AGGREGATE users-idx * GROUPBY 2 @last_login @last_name SORTBY 1 @last_name
```

Now try finding books in the books-idx index published in the year 1983, grouped by author and title, and then sorted by authors and by title.

```bash
FT.AGGREGATE books-idx "@published_year:[1983 1983]" GROUPBY 2 @authors @title SORTBY 2 @authors @title
```
## Reducing

Try counting the number of books in each category to see which categories have the most books.

```bash
FT.AGGREGATE books-idx * GROUPBY 1 @categories REDUCE COUNT 0 AS books_count SORTBY 2 @books_count DESC
```

Now try finding the average rating of all books that mention “tolkien.”

```bash
FT.AGGREGATE books-idx tolkien GROUPBY 0 REDUCE AVG 1 @average_rating as avg_rating
```

## Transforming

Try **finding all books with two co-authors**. 

```bash
FT.AGGREGATE books-idx * APPLY "split(@authors, ';')" AS authors_list GROUPBY 1 @title REDUCE COUNT_DISTINCT 1 authors_list AS authors_count FILTER "@authors_count==2"
```

Now try counting the number of user logins (according to the “last_login” field in the users-idx index) per day of the week. 

```bash
FT.AGGREGATE users-idx * GROUPBY 2 @last_login @user_id APPLY "dayofweek(@last_login)" AS day_of_week GROUPBY 1 @day_of_week REDUCE COUNT 0 AS login_count SORTBY 1 @day_of_week
```

Finally, find the days that more than one user logged in -- in other words, count the distinct IDs of users in the users-idx index who last logged in, grouped by date. You should return the date as a formatted time string, like "2020-12-12T00:00:00Z".

```bash
FT.AGGREGATE users-idx * GROUPBY 2 @last_login @user_id APPLY "day(@last_login)" as last_login_day APPLY "timefmt(@last_login_day)" AS "last_login_str" GROUPBY 1 "@last_login_str" REDUCE COUNT_DISTINCT 1 "@user_id" AS num_logins FILTER "@num_logins>1"
```

---

# 2. Advanced Topics

## Parial Indexes

Imagine that the books-idx index had grown extremely large, and you wanted to split it based on the published year of books. Try creating two indexes: one for books published before 1990 and one for books published on or after 1990.

```bash
FT.CREATE books-older-idx ON HASH PREFIX 1 ru203:book:details: FILTER "@published_year<1990" SCHEMA isbn TAG SORTABLE title TEXT WEIGHT 2.0 SORTABLE subtitle TEXT SORTABLE thumbnail TAG NOINDEX description TEXT SORTABLE published_year NUMERIC SORTABLE average_rating NUMERIC SORTABLE authors TEXT SORTABLE categories TAG SEPARATOR ";" author_ids TAG SEPARATOR ";"

FT.CREATE books-newer-idx ON HASH PREFIX 1 ru203:book:details: FILTER "@published_year>=1990" SCHEMA isbn TAG SORTABLE title TEXT WEIGHT 2.0 SORTABLE subtitle TEXT SORTABLE thumbnail TAG NOINDEX description TEXT SORTABLE published_year NUMERIC SORTABLE average_rating NUMERIC SORTABLE authors TEXT SORTABLE categories TAG SEPARATOR ";" author_ids TAG SEPARATOR ";"
```

Once these indexes exist, try running a couple of queries. For example, get the count of the number of books in each index:

```bash
FT.SEARCH books-older-idx * LIMIT 0 0

FT.SEARCH books-newer-idx * LIMIT 0 0
```

Now try creating an index just for books with the “Fiction” category. Note that when filtering during FT.CREATE, you are filtering on the string values in a Hash.

```bash
FT.CREATE books-fiction-idx ON HASH PREFIX 1 ru203:book:details: FILTER "@categories=='Fiction'" SCHEMA isbn TAG SORTABLE title TEXT WEIGHT 2.0 SORTABLE subtitle TEXT SORTABLE thumbnail TAG NOINDEX description TEXT SORTABLE published_year NUMERIC SORTABLE average_rating NUMERIC SORTABLE authors TEXT SORTABLE categories TAG SEPARATOR ";" author_ids TAG SEPARATOR ";"
```

Now try using an aggregate query to find the authors with the most books published in the books-fiction-idx index.

```bash
FT.AGGREGATE books-fiction-idx * GROUPBY 1 @authors REDUCE COUNT_DISTINCT 1 @title as total_books SORTBY 2 @total_books DESC
```

## Adjusting the Score of a Term 

How might you increase the weight of “greek” books that have the “History” category, while also returning books from other categories?

```bash
FT.SEARCH books-idx "((@categories:{History}) => { $weight: 10 } greek) | greek"
```

Another common boost is to score recent documents in an index higher. How might you use the same technique we used for Greek books to score “cowboy” books higher if they were published after the year 2000?

```bash
FT.SEARCH books-idx "((@published_year:[2000 +inf]) => { $weight: 10 } cowboy) | cowboy"
```