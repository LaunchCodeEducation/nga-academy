---
title: "Walkthrough SwaggerUI"
---

In this walkthrough, the instructor will guide through adding API documentation using [SwaggerUI](https://swagger.io/swagger-ui/).

## Getting Started

### Open Your Launchcart Project in Intellij
The same project and repo you used for the REST studio.
Create and checkout a new branch named `add-swagger` wit this command `git checkout -b add-swagger`

### Setup Intellij

Since we are going to be writing the `swagger.yaml` in Intellij, let's get a plugin to help out.

- Click the magnifying glass in the upper right hand corder and type "Plugin".
- Select "Plugins" from the menu.
- Then select "Browse repositories...".  
- Search for "Swagger" in the search bar and install the "Swagger Plugin".
- Restart Intellij after the plugin has installed.

### Embed SwaggerUI into the Launchcart Project

Clone the [SwaggerUI repository](https://github.com/swagger-api/swagger-ui/tree/2.x) from Github.  Note: We are downloading the SwaggerUI for 2.x.

Navigate into the repo that you just cloned and copy the contents of `swagger-ui/dist` directory into `launchcart/src/main/resources/static/swagger` directory. 

```
  cp -R dist/* {path of launchcart project}/src/main/resources/static/swagger
```

<aside class="aside-note" markdown="1">
  The `dist` directory contains all of the HTML, CSS, and JavaScript required to generate a Swagger document
</aside>

## Setup Swagger .yaml File
In folder `launchcart/src/main/resources/static/swagger`:
1. Create a swagger.yaml file  `touch swagger.yaml`.
2. Edit `swagger\index.html` to point at the local `swagger.yaml` to look like the example below:

```
<script>
window.onload = function() {
  
  // Build a system
  const ui = SwaggerUIBundle({
    url: "http://localhost:8080/swagger/swagger.yaml",
    dom_id: '#swagger-ui',
    deepLinking: true,
    presets: [
      SwaggerUIBundle.presets.apis,
      SwaggerUIStandalonePreset
    ],
    plugins: [
      SwaggerUIBundle.plugins.DownloadUrl
    ],
    layout: "StandaloneLayout"
  })

  window.ui = ui
}
</script>

```
## Load Swagger in the Browser
Start up SpringBoot and navigate to the url `http://localhost:8080/swagger/index.html`.  You should see a SwaggerUI page displayed.

## Writing the Swagger YAML

Next we need to begin writing the Swagger YAML file.  Copy the following code into your `swagger.yaml` file located in the `launchcart/src/main/resources/static/swagger` directory.

```
swagger: '2.0'
info:
  description: |
   This is an example RESTful API 
  version: 1.0.0
  title: LaunchCart API
  termsOfService: http://swagger.io/terms/
  contact:
    email: your.email@gmail.com 
  license:
    name: Apache 2.0
    url: http://www.apache.org/licenses/LICENSE-2.0.html
tags:
paths:
definitions:
```

Let's start with the `/api/carts` path.

Add an entry to the `tags` section, to add a header for all of the endpoints for the `/api/carts` path.

```
- name: cart
	description: Cart provides access to all of the items you are about to buy.
```

Also, let's add the `GET` endpoint for `/api/carts` in the `paths` section.

```
paths:
	/carts:
		get:
			tags:
			- cart
			summary: Returns all carts that exist..
			operationId: getAllCarts
			produces:
			- application/json 
			responses:
				200:
					description: successful operation
```

Next, fill in the schema for the `/api/carts` endpoint.  In order to do that, first check to see what the service is currently returning.

### Review Cart JSON
- Register for an account on your LaunchCart app
- Add item(s) to your cart
- Visit `http://localhost:8080/api/carts`
- You should receive something that looks like the below...

```
[{"uid":1,"items":[{"uid":1,"name":"Chacos","price":1000.0,"newItem":true,"description":"I think they're sandles"}]}]
``` 

To represent this as a schema, add the following to your Swagger config:

```
schema:
	type: object
	required:
	- uid
	- items
	properties:
		uid:
			type: integer
			format: int32
			example: 34
		items:
			type: array
			items:
				type: object
				required:
				- uid
				- name
				- price
				- newItem
				- description
				properties:
					uid:
						type: integer
						format: int32
					name:
						type: string
						example: "Chacos"
					price:
						type: number
						format: int64
						example: 1.00
					newItem:
						type: boolean
						example: true
					description:
						type: string
						example: I think they're a type of sandals
```

<aside class="aside-note" markdown="1">
	Make sure that your whitespace is correct.  There can only be a one tab indent for every map.  
	Incorrect indentation may cause your API endpoints not to show up or display erros.
</aside>

### Yikes!
That was a lot of YAML.  Let's see if we can simplify it and make it less dense.

Move the `items` schema to the the `definitions` section of the yaml config file:

```
definitions:
	Item:
		type: object
		properties:
			uid:
				type: integer
				format: int32
			name:
				type: string
				example: "Chacos"
			price:
				type: number
				format: int64
				example: 1.00
			newItem:
				type: boolean
				example: true
			description:
				type: string
				example: I think they're a type of sandals

```

Replace the `item` schema with a reference to the item in the `definitions` section.

```
	$ref: "#/definitions/Item"
```

## Continue On

Continue on to provide documentation for the rest of the Cart API.  It should be much easier now that the Cart definition has been created.