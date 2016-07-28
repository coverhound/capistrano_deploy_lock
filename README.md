# Capistrano Deploy Lock

Lock a server during deploy, to prevent people from deploying at the same time.


## Installation

Add this line to your application's Gemfile:

    gem 'capistrano-deploy_lock'

And then execute:

    $ bundle

Add this line to your `config/deploy.rb`:

    require 'capistrano/deploy_lock'


## Usage

Your deploys will now be protected by a lock. Simply run `cap deploy` as usual.
However, if someone else tries to deploy at the same time, their deploy will show
a message like this, and exit:

```
*** Deploy locked 3 minutes ago by 'ndbroadbent'
*** Message: Deploying master branch
*** Expires in 7 minutes
```

The default deploy lock will expire after 10 minutes. This is so that crashed or interrupted deploys don't leave a stale lock behind.

The following tasks will be run before deploy:

  * `deploy:check_lock`
    * Checks for an existing deploy lock. Aborts deploy if a lock exists and it wasn't created by you.
  * `deploy:refresh_lock`
    * If you previously created a lock, this task ensures that your lock won't expire before the default expiry time
  * `deploy:create_lock`
    * If no locks already exist, a default lock will be created with the message: `Deploying <branch>`

The following task will be run after deploy:

  * `deploy:unlock`
    * Removes any default deploy locks. If you set a custom lock, it will not be removed at this step.
    * You can remove a custom deploy lock by running `cap deploy:unlock` by itself, or by chaining `deploy:unlock:force` at the end of the command.

## Tasks

### `deploy:with_lock`

Deploy the latest revision with a custom deploy lock. This lock will not be removed at the end of the deploy.

### `deploy:lock`

Sets a custom deploy lock. You will receive two prompts for input:

* **Lock Message:**

Type the reason for the lock. This message will be displayed to any developers who attempt to deploy.

* **Expire lock at? (optional):**

Set an expiry time for the lock. Leave this blank to make the lock last until someone removes it with `cap deploy:unlock`.

If the [chronic](https://github.com/mojombo/chronic) gem is available, you can type
natural language times like `2 hours`, or `tomorrow at 6am`. If not, you must type times in a format that `DateTime.parse()` can handle,
such as `06:30:00` or `2012-12-12 00:00:00`.

### `deploy:unlock`

Removes the deploy lock. Will not remove a custom deploy lock when it is chained after `deploy:lock`.

### `deploy:unlock:force`

Removes any deploy lock, even when chained after `deploy:lock`.

### `deploy:check_lock`

Check if server is locked. If the deploy lock was not created by you, an error will be raised and the deploy will abort.
If the lock **was** created by you, the deploy will pause for 4 seconds, which gives you time to press `Ctrl+C` to cancel the deploy.

This task is also responsible for deleting any expired locks.

### `deploy:refresh_lock`

Refreshes the current lock's expiry time if it is less than the default time.


## Configuration

If your deploys usually take longer than 10 minutes, you can configure the default expiry time with:

```ruby
set :default_lock_expiry, (20 * 60)   # Sets the default expiry to 20 minutes
```

The lock file's default name is `capistrano.lock.yml`. You can change this with:

```ruby
set :deploy_lockfile_name, 'mylockfile.lock.yml'
```

The lock file will be created at `deploy_path.join(fetch(:deploy_lockfile_name))` by default. You can configure this with:

```ruby
set :deploy_lockfile, shared_path.join(fetch(:deploy_lockfile_name))
```


## MOTD script

If you install the gem on your server, you can use the `cap_deploy_lock_msg` script to display the lock status when you login via SSH.

Usage:

    cap_deploy_lock_msg <application_name> <stage> <path/to/lock_file.yml>

If you are using `update-motd` on Ubuntu, create `/etc/update-motd.d/10-deploy-lock` with the following:

    #!/bin/sh
    /usr/local/bin/cap_deploy_lock_msg application stage /var/www/apps/application/shared/capistrano.lock.yml


## Thanks

Special thanks to [David Bock](https://github.com/bokmann), who wrote the [deploy_lock.rb](https://github.com/bokmann/dunce-cap/blob/master/recipes/deploy_lock.rb)
script that this is based on.


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
