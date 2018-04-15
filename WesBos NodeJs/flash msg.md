# Flash Messages

`$ npm install connect-flash`

`app.use(flash());`

### Set up global Variable

Locals are all the variables that are available to you in your templates

`res.locals.flashes = req.flash()`



`req.flash('info', 'Flash is back!')`



in your template layout you set up 

```javasctip
        if locals.flashes
        .inner
          .flash-messages
            - const categories = Object.keys(locals.flashes)
            each category in categories
              each message in flashes[category]
                .flash(class=`flash--${category}`)
                  p.flash__text!= message
                  button.flash__remove(onClick="this.parentElement.remove()") &times;

```

that will only work only if you use sessions

Flash messages are stored in the session. First, setup sessions as usual by enabling `cookieParser` and `session`middleware. Then, use `flash` middle ware provided by connect-flash.