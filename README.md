**GOMODEST** is a Multi Page App(MPA) **starter kit** using Go's `html/template`, `SvelteJS` and `StimulusJS`. It is inspired from modest approaches to building webapps as enlisted in https://modestjs.works/.

## Motivation

I am a devops engineer who dabbles in UI for side projects, internal tools and such. The SPA/ReactJS ecosystem is too costly for me. This is an alternative approach.


> The main idea is to use server rendered html with spots of client-side dynamism using SvelteJS & StimulusJS
 
The webapp is mostly plain old javascript, html, css but with sprinkles of StimulusJS & spots of SvelteJS used for interactivity sans page reloads. StimulusJS is used for sprinkling
interactivity in server rendered html & mounting Svelte components into divs.

## Stack 

A few things which were used:

1. Go, html/template, goview, 
2. Authentication: [github.com/adnaan/authn](https://github.com/adnaan/authn)
3. SvelteJS
4. [Hotwire](https://hotwire.dev/) 
5. Bulma CSS

Many more things in `go.mod` & `web/package.json`

Running
======

Copy the sample env.development to .env.  Edit the .env file with your development settings:

Settings
-----

Configure [Sign in with Google](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid) by creating an OAuth consent screen and an OAuth client.
Put the values of APP_GOOGLE_CLIENT_ID and APP_GOOGLE_SECRET from the creation of
the OAuth client.

Configure the SMTP account settings.  If you are using GMail use these values:
>> APP_SMTP_HOST=smtp.gmail.com
>> APP_SMTP_PORT=587

Set the APP_PLANS_FILE value to the sample `plans.development.json` file.



To run, clone this repo and edit the values in `env.development`

```bash
# terminal 1
$ cd assets && npm install && npm run watch

# terminal 2
$ go run main.co --config env.development
```

The ideas in this starter kit follow the JS gradient as noted [here](https://modestjs.works/book/part-2/the-js-gradient/). I have taken the liberty to organise them into the following big blocks: **server-rendered html**, **sprinkles** and **spots**.

## Server Rendered HTML

Use `html/template` and `goview` to render html pages. It's quite powerful when do you don't need client-side interactions.

example: 

```go
func accountPage(w http.ResponseWriter, r *http.Request) (goview.M, error) {

	session, err := store.Get(r, "auth-session")
	if err != nil {
		return nil, fmt.Errorf("%v, %w", err, InternalErr)
	}

	profileData, ok := session.Values["profile"]
	if !ok {
		return nil, fmt.Errorf("%v, %w", err, InternalErr)
	}

	profile, ok := profileData.(map[string]interface{})
	if !ok {
		return nil, fmt.Errorf("%v, %w", err, InternalErr)
	}

	return goview.M{
		"name": profile["name"],
	}, nil

}
```

## Sprinkles

Use `stimulusjs` to level up server-rendered html to handle simple interactions like: navigations, form validations etc.

example:

```html
    <button class="button is-primary is-outlined is-fullwidth"
            data-action="click->navigate#goto"
            data-goto="/  "
            type="button">
        Home
    </button>
```

```js
    goto(e){
        if (e.currentTarget.dataset.goto){
            window.location = e.currentTarget.dataset.goto;
        }
    }
```

## Spots

Use `sveltejs` to take over `spots` of a server-rendered html page to provide more complex interactivity without page reloads.

This snippet is the most interesting part of this project: 

```html
{{define "content"}}
    <div class="columns is-mobile is-centered">
        <div class="column is-half-desktop">
            <div
                    data-target="svelte.component"
                    data-component-name="app"
                    data-component-props="{{.Data}}">
            </div>
        </div>
    </div>
    </div>
{{end}}
```

[source](https://github.com/adnaan/gomodest-starter/blob/main/web/html/app.html)

In the above snippet, we use StimulusJS to mount a Svelte component by using the following code:

```js
     import { Controller } from "stimulus";
     import components from "./components";
     
     export default class extends Controller {
         static targets = ["component"]
         connect() {
             if (this.componentTargets.length > 0){
                 this.componentTargets.forEach(el => {
                     const componentName = el.dataset.componentName;
                     const componentProps = el.dataset.componentProps ? JSON.parse(el.dataset.componentProps): {};
                     if (!(componentName in components)){
                         console.log(`svelte component: ${componentName}, not found!`)
                         return;
                     }
                     const app = new components[componentName]({
                         target: el,
                         props: componentProps
                     });
                 })
             }
         }
     }
```
[source](https://github.com/adnaan/gomodest-starter/blob/main/web/src/controllers/svelte_controller.js)

This strategy allows us to mix server rendered HTML pages with client side dynamism.

Other possibly interesting aspects could be the layout of [web/html](https://github.com/adnaan/gomodest-starter/tree/main/web/html) and the usage of the super nice [goview](https://github.com/foolin/goview) library to render html in these files: 

 1. [router.go](https://github.com/adnaan/gomodest-starter/blob/main/routes/router.go)
 2. [view.go](https://github.com/adnaan/gomodest-starter/blob/main/routes/view.go)
 3. [pages.go](https://github.com/adnaan/gomodest-starter/blob/main/routes/pages.go)
 
 That is all.
 
 ---------------


 ## Attributions

1. https://areknawo.com/making-a-todo-app-in-svelte/
2. https://modestjs.works/
3. https://github.com/foolin/goview

    

