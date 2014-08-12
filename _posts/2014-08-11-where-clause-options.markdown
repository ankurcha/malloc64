# Thoughts on `where` Clauses for analytics API

The current API where clause is limited in terms of functionality and the set of operations it can support. This document represents a survey of the possible alternatives.

The requirements of any alternative is that it supports (either partially or fully) the following list of features:
* An extensible array of operators, namely:
	* `==`, `!=` Equality and inequality (both point query and 'in' query) operator
	* `<`, `<=`, `>` and `>=` Range query opeators
	* `&&`, `||` and `!` Logical operators
	* `(`, `)` grouping operators

The alternatives below are evaluated on three axes:
* Ease of use
	* A new user should find it easy to read the documentation/examples and not find something in the API very difficult to use. This covers the syntax as well as the readability of the syntax. This requirement also ties into how much effort is needed to on-board a new user in terms of reading the documentation, examples and being able to get started. The effort needed to get from zero to understanding a sizable chuck of the functionality so that the developer may feel productive.
* Extensibility
	* This requirement is more forward facing and intends to capture how easy is it to extend the functionality provided by an option. The developer who creates a new piece of functionality should be able to add that feature without disturbing the ease of use argument.

## Option 1
This option suggests that the URI query string be used like a regular string and we construct queries based on a set of developer defined operators. This means that we could potentially specify a query string that is very readable but not necessarily similar to the key-value syntax of most URI query strings.

### Extensibility
* The developer is expected to maintain a proper parser to ensure that a user request and all the components are parsed into a well defined parse tree. This option may exactly contain the operators as defined above in infix notation.
* There are a lot of options to create a parser, such as antlr or even using regular regex combined with custom logic to interpret the final set of values.

### Ease of use
* In terms of readability the user would find that the query string closely resembles any logical infix expression. This ensures that the syntax is mostly intuitive and after understanding the function of each of the operators they would be able to understand their intended purpose.
* There may be some confusion about escaping the operators and arguments.
* A lot of the http clients out there only support key-value syntax and the user may need to switch or figure out workarounds for this syntax.

### Example:

In this example we use `;`, `,` to denote logical `and` and `or` operations.

`https://api.brightcove.com/v1/accounts/1234567890/report?from=2014-01-04&to=now&offset=200&limit=100&dimensions=video&where=(video_duration%3E=300;video==1,2,3,4;video_view%3C=100),(video_view%3E=9000)`
    
    (video_duration>=300;video==1,2,3,4;video_view<=100),(video_view>=9000)    

The where clause if fairly easy to understand (once decoded) the user knows what the different operators mean but it would still require some getting used to.

* The current Google analytics reporting API v3 is represented in the same way.

## Option 2
This option proposes the use of a grammar that represents a [conjunctive query](http://en.wikipedia.org/wiki/Conjunctive_query) with negation. 
[Parse.com](https://parse.com/docs/rest#queries-basic) does this form of api. As an example consider:

```bash
curl -X GET \
  -H "X-Parse-Application-Id: ${APPLICATION_ID}" \
  -H "X-Parse-REST-API-Key: ${REST_API_KEY}" \
  -G \
  --data-urlencode 'where={"score":{"$gte":1000,"$lte":3000}}' \
  https://api.parse.com/1/classes/GameScore
```

As is evident, here the syntax of the where parameter is a json document that will be url encoded which increases the readability but also makes the user think in a prefix (conjunctive query like) syntax for the expressions.

Another way to represent may use a more familiar syntax by not using json and replacing it with "regular looking" function calls. Eg:

`and(gte(video_duration, 200), or(eq(video, 1), eq(video, 2), eq(video, 3), eq(video_name, foo%2Cbar))`

### Extensibility
* The developer of an this format would need to implement a parser that can pase the expression and then convert it to a corresponding database query.
* Adding new operators would be simple as registering a new `function` name and the corresponding logic needed to generate the database query.

### Ease of use
* As long as the content of the where parameter is properly url encoded, any regualar http client would easily handle the request.
* The user of the API, after learning about the set of functions available would need to think in prefix notation terms and not the regular infix terms. This means that some novice users may find the onboarding process more involved than simply writing a usual logical expression.
