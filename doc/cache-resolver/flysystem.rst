
.. _cache-resolver-flysystem:

Flysystem Resolver
==================

The ``FlysystemResolver`` resolver enables cache resolution using the `Flysystem`_
filesystem abstraction layer.

Dependencies
------------

This cache resolver uses a ``League\\Flysystem\\Filesystem`` to cache files on any source supported
by `Flysystem`_. Flysystem is provided by the ``league/flysystem`` package, but the easiest way to
set up a service is using `The League FlysystemBundle`_ or `OneupFlysystemBundle`_  For simplicity
we show examples for league/flysystem-bundle.

To install the `The League FlysystemBundle`_, run the following composer command:

.. code-block:: bash

    composer require thephpleague/flysystem-bundle

Configuration
-------------

The value of ``filesystem_service`` must be a service id of class ``League\\Flysystem\\Filesystem``.
The service name depends on the naming scheme of the bundle.

```yaml
# config/packages/flysystem.yaml
services:
    Aws\S3\S3Client:
        arguments:
            - version: '2006-03-01'
              credentials:
                  key: '%env(AWS_S3_ACCESS_ID)%'
                  secret: '%env(AWS_S3_ACCESS_SECRET)%'

flysystem:
    storages:
        storage.aws:
            adapter: 'aws'
            options:
                client: 'Aws\S3\S3Client'
                bucket: '%env(AWS_S3_BUCKET_NAME)%'
                prefix: '%env(S3_STORAGE_PREFIX)%'

        storage.local:
            adapter: 'local'
            options:
                directory: '%kernel.project_dir%/var/storage/uploads'

        storage_filesystem:
            adapter: 'lazy'
            options:
                source: '%env(APP_UPLOADS_SOURCE)%'
```

```yaml
# config/packages/liip_imagine.yaml
liip_imagine:
    loaders:
        flysystem_loader:
            flysystem:
                # this comes from flysystem.yaml
                filesystem_service: uploads_filesystem

    # default loader to use for all filter sets
    data_loader: flysystem_loader

    resolvers:
            profile_photos:
                flysystem:
                    filesystem_service: uploads_filesystem
                    root_url:           "https://images.example.com"
                    cache_prefix:       media/cache
                    visibility:         !php/const:League\Flysystem\Visibility::PUBLIC
```

There are several configuration options available:

* ``root_url``: must be a valid url to the target system the flysystem adapter
  points to. This is used to determine how the url should be generated upon request.
  Default value: ``null``
* ``cache_prefix``: this is used for the image path generation. This will be the
  prefix inside the given Flysystem.
  Default value: ``media/cache``
* ``visibility``: one of the two predefined flysystem visibility constants
  (``Visibility::PUBLIC`` / ``Visibility::PRIVATE``
  The visibility is applied, when the objects are stored on a flysystem filesystem.
  You will most probably want to leave the default or explicitly set ``public``.
  Default value: ``public``

Usage
-----

After configuring ``FlysystemResolver``, you can set it as the default cache resolver
for ``LiipImagineBundle`` using the following configuration.

.. code-block:: yaml

    # app/config/config.yml

    liip_imagine:
        cache: profile_photos


Usage on a Specific Filter
~~~~~~~~~~~~~~~~~~~~~~~~~~

Alternatively, you can set it as the cache resolver for a specific filter set using
the following configuration.

```yaml
# config/packages/liip_imagine.yml
liip_imagine:
    filter_sets:
        cache: ~
        my_thumb:
            cache: profile_photos
            filters:
                # the filter list
```


.. _`Flysystem`: https://github.com/thephpleague/flysystem
.. _`OneupFlysystemBundle`: https://github.com/1up-lab/OneupFlysystemBundle
.. _`The League FlysystemBundle`: https://github.com/thephpleague/flysystem-bundle
