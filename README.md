# OLLY

Now in development [join](https://github.com/AtomixInteractions/olly/pulls)

Olly is a framework to built API. Olly configurable by nginx-like config.

```nginx
# for PEG.js

# Define new data model with name Error
model Error {
  prop Number code = 400; # default value. If set to `$status` variable will be resolved from state
  prop String error;
  prop String docs?; # not required property. If provided `false`, `undefined` or `null` was not rendered
}

model User {
  prop Number id?;
  prop String login;
  prop String email;
  prop String password?;
}

model Tag {
  prop String tag;
}

model Post {
  prop Number id;
  prop String title;
  prop String content;
  prop Number author?;
  prop Date created?;
  prop Date modificated?;
  prop Tag[] tags?; # array of custom models
}

model Comment {
  prop Number id;
  prop String content;
  prop Number author;
  prop Date created;
}


# default values for all configuration
host app.lestad.net/api;
scheme https;

# Define API v0
api 0 {
  version path; # You can specify path for version: `version path /first.version`
  # if version is `path` version will search at the end of $host
  # if version is `header`, version will search in header: `version header Accept "application/net.lestad.app.v0"`
  # If version is `none`, version not use. And that loaded if any version not loaded.
  # Default `version path /v$version`

  # host app.lestad.net/api; # you can set specified value for version
  scheme https http; # set protocols. Here is specified scheme for api v0

  mediaType application/json application/xml; # first is used by default if another not set

  # base get, post, put, patch, delete
  # routes must starts with "/"

  get /about; # default resolve to action "about" of controller "index"
  post /register to index; # resolve to action "register" of controller "index"
  put /recover to user@recover; # resolve to action "user" of controller "user".
  # if patch not specified, was created alias for put
  delete /session/:token; # resolve to action "delete" of controller "index" with `token` param

  # extended params
  post /login to user {

    # what model use to filtration and validation
    request User {
      # on request check requiring values
      required [email, login]; # one of property must be sent
      required password; # accept many statements
    }

    # what model use to filtrate for response
    # return: { "user": { "id": 123, "login": "foo", "email": "foo@bar.baz" } }
    response User {
      required id;
    }

    # define error with status and model
    error 403 Error(code= 403, error= "Authorization required"); # Call model with default valus
    error 404 Error(code = 404, error = "User not found");
  }

  get /search {
    deprecated "Use /query"; # If method has `deprecated` statement, router must resolve { "code": 50*, "error": "DEPRECATED", "docs": "Use /query" }
  }

  resources /tags Tag; # resourceS!!! create get, post, put/patch, delete for tags
  # get /tags to tags@index
  # get /tags/:id to tags@show
  # post /tags to tags@create
  # put /tags/:id to tags@update
  # delete /tags/:id to tags@remove
  # All filtration was maked with model `Tag`
  # model for resources is required, name of controller was resolve by model name

  resources /tags Tag {
    # resources can be customized

    # customize default `get /tags to tags@index`
    # can change any value ex.: `get / to tags@all`
    # all paths of inside routes must be related to resource path
    get / {
      response Tag[]; # define what route return list of Tag: { "tags": ["foo", "bar", "baz"] }
    }
  }

  # create part of resource
  # For import to many routes
  concern Commentable {
    resources /comments Comment;
  }

  # define one resource
  # resource can be without model
  resource /back {
    # resource create get, post, put/patch, delete routes
    # for single resource

    get {
      import Commentable; # concern add comments resources to get /back
    }

    # in resource you can specify controller@action without write path
    post to @new; # resolve to back@new
  }

  # /v0/users
  resources /users User {
    # all inside routes will link to /users/:id

    import Commentable;

    # /users/:id/edit
    resource /edit to profile {
      exclude post, delete; # remove routes from resource
    }

    # /users/:id/photo
    resource /photo to userPhoto {
      only post, delete; # create only that routes
    }

    # /users/:id/renew
    post /renew to user;

    # /users/:id/search
    get /search to user;

    # /users/:user_id/tags/:tag
    resources /tags Tags; # one of routes is:
  }
}



# Define API v1
api 1 {
  # will search header `X-API-Version: net.lestad.app.v1` for v1
  version header X-API-Version "net.lestad.app.v$version";

  # scheme was inherited

  mediaType application/json; # use only json. JSON default for Olly

  controller page; # set default controller to defined API

  get /about; # resolve to action "about" of controller "page"
}
```
