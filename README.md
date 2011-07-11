Square-specific things
======================

This is a Square-specific integration branch of Backup.
It mostly tracks meskyanichi/backup:master, but cherry-picks cool stuff from other branches.

To re-generate bundled gems in the submodule, run (with a Ruby VM as close as possible to production):

* bundle package
* bundle install --local --path vendor/bundle


Backup 3
========

Backup is a RubyGem, written for Linux and Mac OSX, that allows you to easily perform backup operations on both your remote, as well as your local environment. It provides you with an elegant DSL in Ruby for modeling (configuring) your backups. Backup has built-in support for various databases, storage protocols/services, syncers, compressors, encryptors and notifiers which you can mix and match. It was built with modularity, extensibility and simplicity in mind.


Author
------

**[Michael van Rooijen](http://michaelvanrooijen.com/) ( [@meskyanichi](http://twitter.com/#!/meskyanichi) )**

Drop me a message for any questions, suggestions, requests, bugs or submit them to the [issue log](https://github.com/meskyanichi/backup/issues).


Installation
------------

To get the latest stable version

    gem install backup

You can view the list of released versions over at [RubyGems.org (Backup)](https://rubygems.org/gems/backup/versions)

Getting Started
---------------

I recommend you read this README first, and refer to the [Wiki pages](https://github.com/meskyanichi/backup/wiki) afterwards. There's also a [Getting Started wiki page](https://github.com/meskyanichi/backup/wiki/Getting-Started).

What Backup 3 currently supports
================================

Below you find a list of components that Backup currently supports. Each of these compontents is isolated, meaning that adding new databases, storage location / service, compressor, encryptor or notifier is easy to do.

Database Support
----------------

- MySQL
- PostgreSQL
- MongoDB
- Redis

[Database Wiki Page](https://github.com/meskyanichi/backup/wiki/Databases)

Filesystem Support
------------------

- Files
- Directories

[Archive Wiki Page](https://github.com/meskyanichi/backup/wiki/Archives)

Storage Locations and Services
------------------------------

- Amazon Simple Storage Service (S3)
- Rackspace Cloud Files (Mosso)
- Dropbox
- Remote Servers *(Available Protocols: FTP, SFTP, SCP and RSync)*

[Storage Wiki Page](https://github.com/meskyanichi/backup/wiki/Storages)

Storage Features
----------------

- Backup Cycling, applies to:
  - Amazon Simple Storage Service (S3)
  - Rackspace Cloud Files (Mosso)
  - Dropbox
  - Remote Servers *(Only Protocols: FTP, SFTP, SCP)*
- Incremental Backups, applies to:
  - Remote Servers *(Only Protocols: RSync)*

[Storage Wiki Page](https://github.com/meskyanichi/backup/wiki/Storages)

Syncers
-------

- RSync
- Amazon Simple Storage Service (S3)

[Syncer Wiki Page](https://github.com/meskyanichi/backup/wiki/Syncers)

Compressors
-----------

- Gzip
- Bzip2

[Compressors Wiki Page](https://github.com/meskyanichi/backup/wiki/Compressors)

Encryptors
----------

- OpenSSL
- GPG

[Encryptors Wiki Page](https://github.com/meskyanichi/backup/wiki/Encryptors)

Notifiers
---------

- Mail
- Twitter
- Campfire
- Presently

[Notifiers Wiki Page](https://github.com/meskyanichi/backup/wiki/Notifiers)

Supported Ruby versions (Tested with RSpec)
-------------------------------------------

- Ruby 1.9.2
- Ruby 1.8.7
- Ruby Enterprise Edition 1.8.7


A sample Backup configuration file
==================================

This is a Backup configuration file. Check it out and read the explanation below. Backup has a [great wiki](https://github.com/meskyanichi/backup/wiki) which explains each component of Backup in detail.

    Backup::Model.new(:sample_backup, 'A sample backup configuration') do

      database MySQL do |database|
        database.name               = 'my_sample_mysql_db'
        database.username           = 'my_username'
        database.password           = 'my_password'
        database.skip_tables        = ['logs']
        database.additional_options = ['--single-transaction', '--quick']
      end

      database MongoDB do |database|
        database.name             = 'my_sample_mongo_db'
        database.only_collections = ['users', 'events', 'posts']
      end

      archive :user_avatars do |archive|
        archive.add '/var/apps/my_sample_app/public/avatars'
      end

      archive :logs do |archive|
        archive.add '/var/apps/my_sample_app/logs/production.log'
        archive.add '/var/apps/my_sample_app/logs/newrelic_agent.log'
        archive.add '/var/apps/my_sample_app/logs/other.log'
      end

      encrypt_with OpenSSL do |encryption|
        encryption.password = 'my_secret_password'
      end

      compress_with Gzip do |compression|
        compression.best = true
      end

      store_with S3 do |s3|
        s3.access_key_id      = 'my_access_key_id'
        s3.secret_access_key  = 'my_secret_access_key'
        s3.region             = 'us-east-1'
        s3.bucket             = 'my_bucket/backups'
        s3.keep               = 20
      end

      sync_with S3 do |s3|
        s3.access_key_id     = "my_access_key_id"
        s3.secret_access_key = "my_secret_access_key"
        s3.bucket            = "my-bucket"
        s3.path              = "/backups"
        s3.mirror            = true

        s3.directories do |directory|
          directory.add "/var/apps/my_app/public/videos"
          directory.add "/var/apps/my_app/public/music"
        end
      end

      notify_by Mail do |mail|
        mail.on_success = false
        mail.on_failure = true
      end

      notify_by Twitter do |tweet|
        tweet.on_success = true
        tweet.on_failure = true
      end

    end

### Brief explanation for the above example configuration

It will dump two databases (MySQL and MongoDB), it'll create two (.t)archives (user_avatars and logs). It'll package the two database and two archives together in a single (.t)archive. It'll run the Gzip compressor to compress that archive, and then it'll run the OpenSSL encryptor to encrypt the compressed archive. Then that encrypted archive will be stored to your Amazon S3 account. If all goes well, and no exceptions are raised, you'll be notified via the Twitter notifier that the backup succeeded. If there was an exception raised during the backup process, then you'd receive an email in your inbox containing detailed exception information, as well as receive a simple Twitter message that something went wrong.

As you can see, you can freely mix and match **archives**, **databases**, **compressors**, **encryptors**, **storages** and **notifiers** for your backups. You could even specify 3 storage locations: Amazon S3, Rackspace Cloud Files and Dropbox, it'd then store your packaged backup to 3 separate locations for high redundancy. This also applies to encryptors, you could double encrypt your backup with OpenSSL followed by GPG if you wanted.

Additionally we have also defined a **S3 Syncer** ( `sync_with S3` ), which does not follow the above process of archiving/compression/encryption, but instead will directly sync the whole `videos` and `music` folder structures from your machine to your Amazon S3 account. (very efficient and cost-effective since it will only transfer files that were changed!)

There are more **archives**, **databases**, **compressors**, **encryptors**, **storages** and **notifiers** than displayed in the example, all available components are listed at the top of this README, as well as in the [Wiki](https://github.com/meskyanichi/backup/wiki).

### Running the example

Notice the `Backup::Model.new(:sample_backup, 'A sample backup configuration') do` at the top of the above example. The `:sample_backup` is called the **trigger**. This is used to identify the backup procedure/file and initialize it.

    backup perform -t [--trigger] sample_backup

Now it'll run the backup, it's as simple as that.

### Automatic backups

Since Backup is a simple command line utility, you should write a crontask to invoke it periodically. I recommend you use [Whenever](https://github.com/javan/whenever) to manage your crontab. It'll allow you to write to the crontab in pure Ruby, it provides an elegant DSL to do so, for example:

    every 6.hours do
      command "backup perform --trigger sample_backup"
    end

With this in place, run `whenever --update-crontab backup` to write this Ruby syntax to the crontab in cron-syntax. The operating system will now invoke `backup perform --trigger sample_backup` every 6 hours. Check out the Whenever project page for more information.

Documentation
-------------

See the [Wiki Pages](https://github.com/meskyanichi/backup/wiki). The subjects labeled **without** the "Backup 2)"-prefix are meant for Backup 3 users.


Suggestions, Bugs, Requests, Questions
--------------------------------------

View the [issue log](https://github.com/meskyanichi/backup/issues) and post them there.

Contributors
------------

<table>
  <tr>
    <th>Contributor</th>
    <th>Contribution</th>
  </tr>
  <tr>
    <td><a href="https://github.com/asanghi" target="_blank">Aditya Sanghi ( asanghi )</a></td>
    <td>Twitter Notifier, Dropbox Timeout Configuration</td>
  </tr>
  <tr>
    <td><a href="https://github.com/phlipper" target="_blank">Phil Cohen ( phlipper )</a></td>
    <td>Exclude Option for Archives</td>
  </tr>
  <tr>
    <td><a href="https://github.com/arunagw" target="_blank">Arun Agrawal ( arunagw )</a></td>
    <td>Campfire notifier</td>
  </tr>
  <tr>
    <td><a href="https://github.com/szimmermann" target="_blank">Stefan Zimmermann ( szimmermann )</a></td>
    <td>Enabling package/archive (tar utility) support for more Linux distro's (FreeBSD, etc)</td>
  </tr>
  <tr>
    <td><a href="https://github.com/trystant" target="_blank">Mark Nyon ( trystant )</a></td>
    <td>Helping discuss MongoDump Lock/FSync problem</td>
  </tr>
  <tr>
    <td><a href="https://github.com/imanel" target="_blank">Bernard Potocki ( imanel )</a></td>
    <td>Helping discuss MongoDump Lock/FSync problem + Submitting a patch</td>
  </tr>
  <tr>
    <td><a href="https://github.com/tomash" target="_blank">Tomasz Stachewicz ( tomash )</a></td>
    <td>Helping discuss MongoDump Lock/FSync problem + Submitting a patch</td>
  </tr>
  <tr>
    <td><a href="https://github.com/lapluviosilla" target="_blank">Paul Strong ( lapluviosilla )</a></td>
    <td>Helping discuss MongoDump Lock/FSync problem</td>
  </tr>
  <tr>
    <td><a href="https://github.com/rgnitz" target="_blank">Ryan ( rgnitz )</a></td>
    <td>Helping discuss MongoDump Lock/FSync problem</td>
  </tr>
  <tr>
    <td><a href="https://github.com/tsigo" target="_blank">Robert Speicher ( tsigo )</a></td>
    <td>Adding the --quiet [-q] feature to Backup to silence console logging</td>
  </tr>
  <tr>
    <td><a href="https://github.com/jwhitcraft" target="_blank">Jon Whitcraft ( jwhitcraft )</a></td>
    <td>Adding the ability to add additional options to the S3Syncer</td>
  </tr>
  <tr>
    <td><a href="https://github.com/bgarret" target="_blank">Benoit Garret ( bgarret )</a></td>
    <td>Presently notifier</td>
  </tr>
</table>


Want to contribute?
-------------------

- Fork/Clone the **develop** branch
- Write RSpec tests, and test against:
  - Ruby 1.9.2
  - Ruby 1.8.7
  - Ruby Enterprise Edition 1.8.7
- Try to keep the overall *structure / design* of the gem the same

I can't guarantee I'll pull every pull request. Also, I may accept your pull request and drastically change parts to improve readability/maintainability. Feel free to discuss about improvements, new functionality/features in the [issue log](https://github.com/meskyanichi/backup/issues) before contributing if you need/want more information.

Easily run tests against all three Ruby versions
------------------------------------------------

Install [RVM](https://rvm.beginrescueend.com/) and use it to install Ruby 1.9.2, 1.8.7 and REE.

    rvm install 1.9.2 && rvm install 1.8.7 && rvm install ree

Once these are installed, go ahead and install all the necessary dependencies.

    cd backup
    rvm use 1.9.2 && gem install bundler && bundle install
    rvm use 1.8.7 && gem install bundler && bundle install
    rvm use ree   && gem install bundler && bundle install

The Backup gem uses [Guard](https://github.com/guard/guard) along with [Guard::RSpec](https://github.com/guard/guard-rspec) to quickly and easily test Backup's code against all three Rubies. If you've done the above, all you have to do is run:

    guard start

from Backup's root and that's it. It'll now test against all three Rubies each time you adjust a file in the `lib` or `spec` directories.


Or contribute by writing blogs/tutorials
----------------------------------------

- http://erik.debill.org/2011/03/26/csing-backup-with-rails
- http://blog.noizeramp.com/2011/03/31/backing-up-backup-ruby-gem/
- http://www.sebaugereau.com/using-ruby-to-backup-with-beauty
- http://outofti.me/post/4159686269/backup-with-pgbackups
- http://h2ik.co/2011/03/backing-up-with-ruby/


Backup 2 - Issues, Wiki, Source, Gems
=====================================

I won't actively support Backup 2 anymore. The source will remain on [a separate branch](https://github.com/meskyanichi/backup/tree/backup-2). [The Issues](https://github.com/meskyanichi/backup/issues) that belong to Backup 2 have been tagged with a black label "Backup 2". The Backup 2 specific [Wiki pages](https://github.com/meskyanichi/backup/wiki) have been prefixed with "Backup 2) <Article>". [The Backup 2 Gems](http://rubygems.org/gems/backup) will always remain so you can still use Backup 2. I might still accept pull requests, but would highly encourage anyone to [move to __Backup 3__ once it's here](https://github.com/meskyanichi/backup).
