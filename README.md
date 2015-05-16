
## Installation

```sh
$ npm install sinopia
$ npm install sinopia-htpasswd
```

PS: Actually, this module is bundled with sinopia, so you don't have to install it like this. But with other auth plugins you have to.

## Config

Add to your `config.yaml`:

```yaml
auth:
  htpasswd:
    file: ./htpasswd
    group_file: ./htpgroup

    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    #max_users: 1000
```

## Password file format

Passwords are stored in `htpasswd` file, and it is designed to be compatible with Nginx.

We use standard password hashing in linux (same as in `/etc/shadow`, on most modern systems it's salted `SHA-512`), but you can use whatever your glibc supports.

You can manually add new user with this command (see different commands [here](http://unix.stackexchange.com/questions/52108/how-to-create-sha512-password-hashes-on-command-line)):

```sh
$ echo "user:`mkpasswd --method=sha-512 password`" >> htpasswd
```

## Group file format

The group file `htgroup` is optionnal. It's useful to make group of users.

Example of `htgroup` file:
```
publisher: user1 user2 user3
```

And in your `config.yaml`:
```yaml
packages:
  'local-*':
    allow_access: $authenticated
    allow_publish: publisher
```

## For plugin writers

It's called as:

```js
require('sinopia-htpasswd')(config, stuff)
```

Where:

 - config - module's own config
 - stuff - collection of different internal sinopia objects
   - stuff.config - main config
   - stuff.logger - logger

This should export two functions:

 - `adduser(user, password, cb)`
   
   It should respond with:
    - `cb(err)` in case of an error (error will be returned to user)
    - `cb(null, false)` in case registration is disabled (next auth plugin will be executed)
    - `cb(null, true)` in case user registered successfully
   
   It's useful to set `err.status` property to set http status code (e.g. `err.status = 403`).

 - `authenticate(user, password, cb)`
   
   It should respond with:
    - `cb(err)` in case of a fatal error (error will be returned to user, keep those rare)
    - `cb(null, false)` in case user not authenticated (next auth plugin will be executed)
    - `cb(null, [groups])` in case user is authenticated
   
   Groups is an array of all users/usergroups this user has access to. You should probably include username itself here.
   
