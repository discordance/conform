# Conform - guards Go structs against user input (golang)

Trim, sanitize, and modify struct string fields in place, based on tags.

Turns this...

``` go
type Person struct {
	FirstName string `conform:"name"`
	LastName  string `conform:"ucfirst,trim"`
	Email     string `conform:"email"`
	CamelCase string `conform:"camel"`
	UserName  string `conform:"snake"`
	Slug      string `conform:"slug"`
	Blurb     string `conform:"title"`
	Left      string `conform:"ltrim"`
	Right     string `conform:"rtrim"`
}

p1 := Person{
	" LEE ",
	"     Benson",
	"   LEE@LEEbenson.com  ",
	"I love new york city",
	"lee benson",
	"LeeBensonWasHere",
	"this is a little bit about me...",
	"    Left trim   ",
	"    Right trim  ",
}

```

Into this...

``` go
p2 := p1 // <-- copy the Person struct into a new one, to see the difference
conform.Strings(&p2) // <-- this does the work

/*
	p1 (left) vs. p2 (right)

	FirstName: ' LEE ' -> 'Lee'
	LastName: '     Benson' -> 'Benson'
	Email: '   LEE@LEEbenson.com  ' -> 'lee@leebenson.com'
	CamelCase: 'I love new york city' -> 'ILoveNewYorkCity'
	UserName: 'lee benson' -> 'lee_benson'
	Slug: 'LeeBensonWasHere' -> 'lee-benson-was-here'
	Blurb: 'this is a little bit about me...' -> 'This Is A Little Bit About Me...'
	Left: '    Left trim   ' -> 'Left trim   '
	Right: '    Right trim  ' -> '    Right trim'
*/
```

## Why?

Conform helps you fix and format user strings quickly, without writing functions.

If you do form processing with [Gorilla Schema](http://www.gorillatoolkit.org/pkg/schema) or similar, you probably shuttle user data into structs using tags. Adding a `conform` tag to your string field gives you "first pass" clean up against user input.

Use it for names, e-mail addresses, URL slugs, or any other form field where formatting matters.

Conform doesn't attempt any kind of validation on your fields. Check out [govalidator](https://github.com/asaskevich/govalidator).

## How to use

Grab the package from the command line with:

`go get github.com/leebenson/conform`

And import in the usual way in your Go app:

`import "github.com/leebenson/conform"`

Add a `conform` tag to your structs, for all of the string fields that you want Conform to transform. Add the name of the transform (known as the "tag") in double quotes, and separate multiple tags with commas. Example: `conform:"trim,lowercase"`

To format in place, pass your struct pointer to `conform.Strings`.

**Note: your struct will be edited _in place_. This will OVERWRITE any data that is already stored in your string fields.**

Here's an example that formats e-mail addresses:

``` go
package main

import (
		"fmt"
		"github.com/leebenson/conform"
)

type UserForm struct {
	Email string `conform:"email"`
}

func main() {
	input := UserForm{
		Email: "   POORLYFormaTTED@EXAMPlE.COM  ",
	}
	conform.Strings(&input) // <-- pass in a pointer to your struct
	fmt.Println(input.Email) // prints "poorlyformatted@example.com"
}

```

## Using with Gorilla Schema

Just add a `conform` tag along with your Gorilla `schema` tags:

``` go
// ...

import (
	"net/http"

	"github.com/gorilla/schema"
	"github.com/leebenson/conform"
)

// the struct that will be filled from the post request...
type newUserForm struct {
	FirstName string `schema:"firstName",conform:"name"`
	Email     string `schema:"emailAddress",conform:"email"`
	Password  string `schema:"password"`    // <-- no change? no tag
	Dob       string `schema:"dateOfBirth"` // <-- non-strings ignored by conform
}

// ProcessNewUser attempts to register a new user
func ProcessNewUser(r *http.Request) error {
	form := new(newUserForm)
	schema.NewDecoder().Decode(form, r.PostForm) // <-- Gorilla Schema
	conform.Strings(form)                       // <-- Conform.  Pass in the same pointer that Schema used
	// ...
}

// HTTP handlers, etc...

```

## Godoc
See the [public API / exported methods on Godoc](https://godoc.org/github.com/leebenson/conform).

## Tags

You can use multiple tags in the format of `conform:"tag1,tag2"`

### trim
---------------------------------------
Trims leading and trailing spaces. Example: `"   string   "` -> `"string"`

### ltrim
---------------------------------------
Trims leading spaces only. Example: `"   string   "` -> `"string   "`

### rtrim
---------------------------------------
Trims trailing spaces only. Example: `"   string   "` -> `"   string"`

### lower
---------------------------------------
Converts string to lowercase. Example: `"STRING"` -> `"string"`

### upper
---------------------------------------
Converts string to uppercase. Example: `"string"` -> `"STRING"`

### title
---------------------------------------
Converts string to Title Case, e.g. `"this is a sentence"` -> `"This Is A Sentence"`

### camel
---------------------------------------
Converts to camel case via [stringUp](https://github.com/etgryphon/stringUp), Example provided by library: `this is it => thisIsIt, this\_is\_it => thisIsIt, this-is-it => thisIsIt`

### snake
---------------------------------------
Converts to snake_case. Example: `CamelCase` -> `camel_case`, `regular string` -> `regular_string`
Special thanks to [snaker](https://github.com/serenize/snaker/) for inspiration (credited in license)

### slug
---------------------------------------
Turns strings into slugs.  Example: `CamelCase` -> `camel-case`, `blog title here` -> `blog-title-here`

### ucfirst
---------------------------------------
Uppercases first character.  Example: `all lower` -> `All lower`

### name
---------------------------------------
Trims, uppercases the first character, lowercases the rest. Example: ` JOHN ` -> `John`

### email
---------------------------------------
Trims and lowercases the string.  Example: `UNSIGHTLY-EMAIL@EXamPLE.com ` -> `unsightly-email@example.com`

### LICENSE
[MIT](https://github.com/leebenson/conform/blob/master/LICENSE)
