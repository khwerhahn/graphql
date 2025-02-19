graphql for Sorare.com
=======
Package `graphql` provides a GraphQL client implementation to cater for a sorare.com API Key

For more information, see package [`github.com/khwerhahn/githubv4`](https://github.com/khwerhahn/githubv4), which is a specialized version targeting GitHub GraphQL API v4. That package is driving the feature development.

Installation
------------

`graphql` requires Go version 1.8 or later.

```bash
go get -u github.com/khwerhahn/graphql
```

Usage
-----

This package was forked !!just!! to work for a sorare.com project. 

You have to inlcude an environment variable ```SORARE_API_KEY``` to work. Make sure that you throttle your API calls according to the latest sorare docs. 


Construct a GraphQL client, specifying the GraphQL server URL. Then, you can use it to make GraphQL queries and mutations.

```Go
client := graphql.NewClient("https://api.sorare.com/graphql")
// Use client...
```

### Authentication

Sorare API requires a header ```APIKEY```. This package adds it automatically via the environment variable ```SORARE_API_KEY```. Make that you add it. 


### Sorare API
Please follow the official [sorare.com](https://github.com/sorare/api) documentation. There is also a [graphql playground](https://api.sorare.com/graphql/playground).
### General Queries

To make a GraphQL query, you need to define a corresponding Go type.

For example, to make the following GraphQL query:

```GraphQL
query {
	me {
		name
	}
}
```

You can define this variable:

```Go
var query struct {
	Me struct {
		Name graphql.String
	}
}
```

Then call `client.Query`, passing a pointer to it:

```Go
err := client.Query(context.Background(), &query, nil)
if err != nil {
	// Handle error.
}
fmt.Println(query.Me.Name)

// Output: Luke Skywalker
```

### Arguments and Variables

Often, you'll want to specify arguments on some fields. You can use the `graphql` struct field tag for this.

For example, to make the following GraphQL query:

```GraphQL
{
	human(id: "1000") {
		name
		height(unit: METER)
	}
}
```

You can define this variable:

```Go
var q struct {
	Human struct {
		Name   graphql.String
		Height graphql.Float `graphql:"height(unit: METER)"`
	} `graphql:"human(id: \"1000\")"`
}
```

Then call `client.Query`:

```Go
err := client.Query(context.Background(), &q, nil)
if err != nil {
	// Handle error.
}
fmt.Println(q.Human.Name)
fmt.Println(q.Human.Height)

// Output:
// Luke Skywalker
// 1.72
```

However, that'll only work if the arguments are constant and known in advance. Otherwise, you will need to make use of variables. Replace the constants in the struct field tag with variable names:

```Go
var q struct {
	Human struct {
		Name   graphql.String
		Height graphql.Float `graphql:"height(unit: $unit)"`
	} `graphql:"human(id: $id)"`
}
```

Then, define a `variables` map with their values:

```Go
variables := map[string]interface{}{
	"id":   graphql.ID(id),
	"unit": starwars.LengthUnit("METER"),
}
```

Finally, call `client.Query` providing `variables`:

```Go
err := client.Query(context.Background(), &q, variables)
if err != nil {
	// Handle error.
}
```

### Inline Fragments

Some GraphQL queries contain inline fragments. You can use the `graphql` struct field tag to express them.

For example, to make the following GraphQL query:

```GraphQL
{
	hero(episode: "JEDI") {
		name
		... on Droid {
			primaryFunction
		}
		... on Human {
			height
		}
	}
}
```

You can define this variable:

```Go
var q struct {
	Hero struct {
		Name  graphql.String
		Droid struct {
			PrimaryFunction graphql.String
		} `graphql:"... on Droid"`
		Human struct {
			Height graphql.Float
		} `graphql:"... on Human"`
	} `graphql:"hero(episode: \"JEDI\")"`
}
```

Alternatively, you can define the struct types corresponding to inline fragments, and use them as embedded fields in your query:

```Go
type (
	DroidFragment struct {
		PrimaryFunction graphql.String
	}
	HumanFragment struct {
		Height graphql.Float
	}
)

var q struct {
	Hero struct {
		Name          graphql.String
		DroidFragment `graphql:"... on Droid"`
		HumanFragment `graphql:"... on Human"`
	} `graphql:"hero(episode: \"JEDI\")"`
}
```

Then call `client.Query`:

```Go
err := client.Query(context.Background(), &q, nil)
if err != nil {
	// Handle error.
}
fmt.Println(q.Hero.Name)
fmt.Println(q.Hero.PrimaryFunction)
fmt.Println(q.Hero.Height)

// Output:
// R2-D2
// Astromech
// 0
```

### Mutations

Mutations often require information that you can only find out by performing a query first. Let's suppose you've already done that.

For example, to make the following GraphQL mutation:

```GraphQL
mutation($ep: Episode!, $review: ReviewInput!) {
	createReview(episode: $ep, review: $review) {
		stars
		commentary
	}
}
variables {
	"ep": "JEDI",
	"review": {
		"stars": 5,
		"commentary": "This is a great movie!"
	}
}
```

You can define:

```Go
var m struct {
	CreateReview struct {
		Stars      graphql.Int
		Commentary graphql.String
	} `graphql:"createReview(episode: $ep, review: $review)"`
}
variables := map[string]interface{}{
	"ep": starwars.Episode("JEDI"),
	"review": starwars.ReviewInput{
		Stars:      graphql.Int(5),
		Commentary: graphql.String("This is a great movie!"),
	},
}
```

Then call `client.Mutate`:

```Go
err := client.Mutate(context.Background(), &m, variables)
if err != nil {
	// Handle error.
}
fmt.Printf("Created a %v star review: %v\n", m.CreateReview.Stars, m.CreateReview.Commentary)

// Output:
// Created a 5 star review: This is a great movie!
```

Directories
-----------

| Path                                                                                   | Synopsis                                                                                                        |
|----------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [example/graphqldev](https://godoc.org/github.com/khwerhahn/graphql/example/graphqldev) | graphqldev is a test program currently being used for developing graphql package.                               |
| [ident](https://godoc.org/github.com/khwerhahn/graphql/ident)                           | Package ident provides functions for parsing and converting identifier names between various naming convention. |
| [internal/jsonutil](https://godoc.org/github.com/khwerhahn/graphql/internal/jsonutil)   | Package jsonutil provides a function for decoding JSON into a GraphQL query data structure.                     |

License
-------

-	[MIT License](LICENSE)
